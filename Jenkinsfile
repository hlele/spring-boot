pipeline {
    agent {
        any
    }

    tools {
        // Specify the JDK and Gradle versions configured in Global Tool Configuration
        // Make sure these names match exactly what you defined there.
        jdk 'JDK_21' // Example: Name of your JDK installation in Jenkins
        gradle 'Gradle_8.x' // Example: Name of your Gradle installation in Jenkins
    }

    environment {
        // SonarQube server details (configured in Jenkins global tools or credentials)
        SONAR_SCANNER_HOME = tool 'SonarQube Scanner' // Name of your SonarQube Scanner installation in Jenkins
        SONAR_QUBE_URL = 'http://localhost:9000' // Replace with your SonarQube server URL
        SONAR_QUBE_CREDENTIALS_ID = 'Jenkins-SQ-Analysis' // ID of your SonarQube token credential in Jenkins
        // SonarQube project key - unique identifier for your project in SonarQube
        // It's good practice to derive this from repo name or define explicitly.
        //SONAR_PROJECT_KEY = "${env.JOB_NAME.replace('/', '_').toLowerCase()}" // Example: Uses Jenkins job name, converted to lowercase
        SONAR_PROJECT_KEY = "github-spring-boot"

        // SonarQube project name (how it appears in SonarQube UI)
        SONAR_PROJECT_NAME = "github-spring-boot"

        // Path to the Gradle wrapper (usually just 'gradlew')
        GRADLE_WRAPPER = "./gradlew"

        // Default branch for analysis (often 'main' or 'master')
        SONAR_BRANCH_NAME = "hlele-patch-1" // Or "${env.BRANCH_NAME}" if using multi-branch pipeline
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Adjust this for your specific GitHub repository and credentials
                git branch: env.SONAR_BRANCH_NAME,
                    credentialsId: 'github-token-1', // Replace with your GitHub credential ID in Jenkins
                    url: 'https://github.com/hlele/spring-boot.git' // Replace with your actual repository URL
                // For public repos, 'credentialsId' might not be needed.
            }
        }

  //      stage('Build and Test') {
     //       steps {
                // Clean build directory, build the project, and run tests
                // -x test: Exclude tests from the build if you want to run them separately or rely on SonarQube to analyze test results later.
                // It's usually better to run tests as part of your build process.
                sh "${GRADLE_WRAPPER} clean build"
                // If you want to skip tests during the build, but ensure coverage is still generated:
                // sh "${GRADLE_WRAPPER} clean build -x test"
  //          }
   //     }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // This section uses the SonarQube Scanner for Jenkins plugin.
                    // The 'withSonarQubeEnv' block injects SonarQube environment variables.
                    // The name 'MySonarQubeServer' must match the name configured in Jenkins Global System Config.
                    withSonarQubeEnv('Sonarqube-local-docker') { // Name of your SonarQube server config in Jenkins
                        // Run SonarQube analysis using the Gradle SonarQube plugin
                        // The 'sonar' task is typically provided by applying 'org.sonarqube' plugin in build.gradle.
                        sh """${GRADLE_WRAPPER} sonarqube \\
                            -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \\
                            -Dsonar.projectName=${env.SONAR_PROJECT_NAME} \\
                            -Dsonar.host.url=${env.SONAR_QUBE_URL} \\
                            -Dsonar.login=${env.SONAR_QUBE_CREDENTIALS_ID} \\
                            -Dsonar.sourceEncoding=UTF-8 \\
                            -Dsonar.sources=src/main/java,src/main/kotlin \\
                            -Dsonar.tests=src/test/java,src/test/kotlin \\
                            -Dsonar.java.binaries=build/classes \\
                            -Dsonar.junit.reportPaths=build/test-results/test \\
                            -Dsonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml"""
                        // Important notes for properties:
                        // - Dsonar.host.url and -Dsonar.login are automatically provided by withSonarQubeEnv,
                        //   but explicitly passing them here can sometimes resolve issues if env vars aren't propagated.
                        // - Dsonar.sources, Dsonar.tests, Dsonar.java.binaries: Adjust these paths based on your project structure.
                        // - Dsonar.junit.reportPaths: Ensure this points to your JUnit XML reports.
                        // - Dsonar.coverage.jacoco.xmlReportPaths: Make sure your build generates Jacoco XML reports
                        //   (you need to apply 'jacoco' plugin in build.gradle and configure it to generate XML reports).
                    }
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    // This step will wait for the SonarQube analysis to complete and check the Quality Gate status.
                    // It will fail the Jenkins pipeline if the Quality Gate fails in SonarQube.
                    // The name 'MySonarQubeServer' must match the name configured in Jenkins Global System Config.
                    // You can specify a timeout in minutes.
                    timeout(time: 10, unit: 'MINUTES') { // Increased timeout for larger projects
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }

    post {
        always {
            echo "SonarQube scan pipeline finished for ${env.SONAR_PROJECT_NAME}."
        }
        failure {
            echo "SonarQube scan failed or Quality Gate didn't pass for ${env.SONAR_PROJECT_NAME}."
            // Optionally, send notifications
        }
        success {
            echo "SonarQube scan completed and Quality Gate passed for ${env.SONAR_PROJECT_NAME}!"
            // Optionally, send notifications
        }
    }
}

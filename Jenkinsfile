pipeline {
    agent any
    
    environment {
        SONAR_SCANNER_HOME = tool 'SonarQube' // Matches the name in Jenkins Global Tool Config
    }

    stages {
        stage('Step 1: Cleanup & Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Step 2: SonarQube Static Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Matches name in System Config
                    sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=online-boutique \
                    -Dsonar.projectName=online-boutique \
                    -Dsonar.sources=."
                }
            }
        }

        stage('Step 3: Quality Gate Check') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Step 4: Trivy Filesystem Scan') {
            steps {
                sh "trivy fs --severity HIGH,CRITICAL . > trivy-fs-report.txt"
                archiveArtifacts artifacts: 'trivy-fs-report.txt', allowEmptyArchive: true
            }
        }

        stage('Step 5: Trivy Image Scan (Frontend)') {
            steps {
                // Scanning the public frontend image as an example
                sh "trivy image --severity HIGH,CRITICAL gcr.io/google-samples/microservices-demo/frontend:v0.10.1 > trivy-image-report.txt"
                archiveArtifacts artifacts: 'trivy-image-report.txt', allowEmptyArchive: true
            }
        }
    }
}

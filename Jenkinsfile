pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compilation') {
            steps {
                sh './gradlew compileJava'
            }
        }

        stage('Test unitaire') {
            steps {
                sh './gradlew test'
            }
        }

        stage('Couverture de code') {
            steps {
                sh './gradlew jacocoTestReport'
                publishHTML(target: [
                    reportName: 'Rapport JaCoCo',
                    reportDir: 'build/reports/jacoco/test/html',
                    reportFiles: 'index.html'
                ])
            }
        }

        stage('Analyse statique du code') {
            steps {
                sh './gradlew checkstyleMain'
                publishHTML(target: [
                    reportName: 'Checkstyle Report',
                    reportDir: 'build/reports/checkstyle',
                    reportFiles: 'main.html'
                ])
            }
        }

        stage('Package') {
            steps {
                sh './gradlew build'
            }
        }

        stage('Docker build') {
            steps {
                sh "docker build -t localhost:5000/calculatrice ."
            }
        }

        stage('Docker push') {
            steps {
                sh "docker push localhost:5000/calculatrice"
            }
        }

        stage("Deploy to staging ou déployer en préproduction") {
            steps {
                sh "docker rm -f calculatrice || true"
                sh "docker run -d -p 8882:8081 --name calculatrice localhost:5000/calculatrice"
            }
        }

        stage("Acceptance test") {
            steps {
                sleep 60
                // Use bash explicitly to run the acceptance test script
                sh "chmod +x acceptance_test.sh && bash ./acceptance_test.sh"
            }
        }
    }

    post {
        always {
            script {
                try {
                    // Check if the container exists before stopping/removing
                    sh "docker ps -q -f name=calculatrice || true"  // This checks if the container is running
                    sh "docker stop calculatrice || true"
                    sh "docker rm calculatrice || true"
                } catch (Exception e) {
                    echo "Aucun conteneur Docker à arrêter"
                }
            }
        }
    }
}


pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-flask-app'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker Installation') {
            steps {
                bat 'docker --version'  // Check if Docker is installed
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    // Explicitly pass variables and check if Docker build succeeds
                    def buildStatus = bat(script: 'docker build -t my-flask-app:latest .', returnStatus: true)
                    if (buildStatus != 0) {
                        error 'Docker build failed!'
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub (ensure credentials are set in Jenkins credentials store)
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat "echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin"
                    }

                    // Push the Docker image to Docker Hub
                    echo 'Pushing Docker image to Docker Hub...'
                    def pushStatus = bat(script: 'docker push my-flask-app:latest', returnStatus: true)
                    if (pushStatus != 0) {
                        error 'Docker push failed!'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run the Docker container
                    echo 'Running Docker container...'
                    def runStatus = bat(script: 'docker run -d my-flask-app:latest', returnStatus: true)
                    if (runStatus != 0) {
                        error 'Docker container failed to run!'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}

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

        stage('Install Docker') {
            steps {
                script {
                    // Check if Docker is installed and install if necessary
                    def dockerInstalled = sh(script: 'docker --version', returnStatus: true)
                    if (dockerInstalled != 0) {
                        echo 'Docker not installed, attempting to install Docker...'
                        // You can install Docker here if it's not present
                        // For Windows, manual installation or script may be required, as automated installation might not work perfectly through Jenkins.
                        echo 'Install Docker manually from: https://www.docker.com/get-started'
                    } else {
                        echo 'Docker is already installed.'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Make sure the docker build command works on Windows
                    echo 'Building Docker image...'
                    def buildStatus = sh(script: 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .', returnStatus: true)
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
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }

                    // Push the Docker image to Docker Hub
                    echo 'Pushing Docker image to Docker Hub...'
                    def pushStatus = sh(script: 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}', returnStatus: true)
                    if (pushStatus != 0) {
                        error 'Docker push failed!'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo 'Running Docker container...'
                    def runStatus = sh(script: 'docker run -d ${DOCKER_IMAGE}:${DOCKER_TAG}', returnStatus: true)
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

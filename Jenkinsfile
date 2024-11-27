pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-flask-app'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Check Docker Installation') {
            steps {
                script {
                    // Check if Docker is installed
                    def dockerCheck = bat(script: 'docker --version', returnStatus: true)
                    if (dockerCheck != 0) {
                        echo 'Docker is not installed. Installing Docker...'
                        // Docker installation steps for Windows
                        bat '''
                        powershell -Command "Invoke-WebRequest -Uri https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe -OutFile DockerDesktopInstaller.exe"
                        start /wait DockerDesktopInstaller.exe install
                        del DockerDesktopInstaller.exe
                        '''
                        echo 'Docker installation completed. Please restart the agent and rerun the pipeline.'
                        error('Pipeline stopped because Docker was just installed. Restart the agent and rerun.')
                    } else {
                        echo 'Docker is already installed.'
                    }
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                checkout scm
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
                    // Ensure credentials are injected and available
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo "Using Docker username: ${DOCKER_USERNAME}"

                        // Login to Docker Hub (correctly inject username and password)
                        bat "docker logout"
                        bat "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
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

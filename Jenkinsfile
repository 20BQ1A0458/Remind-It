pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-flask-app1'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Check Docker Installation') {
            steps {
                script {
                    def dockerCheck = bat(script: 'docker --version', returnStatus: true)
                    if (dockerCheck != 0) {
                        echo 'Docker is not installed. Installing Docker...'
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
                    def buildStatus = bat(script: "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .", returnStatus: true)
                    if (buildStatus != 0) {
                        error 'Docker build failed!'
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-c', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo 'Logging in to Docker Hub...'

                        // Prevent password from being printed in logs
                        def loginStatus = bat(
                            script: """
                            echo | docker login -u "%DOCKER_USERNAME%" --password-stdin < nul
                            """, 
                            returnStatus: true
                        )

                        if (loginStatus != 0) {
                            error 'Docker login failed!'
                        }

                        echo 'Pushing Docker image to Docker Hub...'
                        def pushStatus = bat(script: "docker push %DOCKER_USERNAME%/${DOCKER_IMAGE}:${DOCKER_TAG}", returnStatus: true)
                        if (pushStatus != 0) {
                            error 'Docker push failed!'
                        }
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo 'Running Docker container...'
                    def runStatus = bat(script: "docker run -d --name my-flask-container -p 8080:80 ${DOCKER_IMAGE}:${DOCKER_TAG}", returnStatus: true)
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

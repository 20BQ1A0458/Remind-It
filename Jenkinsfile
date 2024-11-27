pipeline {
    agent any

    environment {
        // Define Docker Hub credentials and repository
        DOCKER_CREDENTIALS = 'dockerhub-creds'  // Jenkins credential ID for Docker Hub
        DOCKER_IMAGE_NAME = 'bhargavram458/my-flask-app'  // Replace with your Docker image name

        // Define GitHub repository URL (update with your GitHub repo URL)
        GITHUB_REPO_URL = 'https://github.com/20BQ1A0458/Remind-It.git'  // GitHub repository URL
        GITHUB_CREDENTIALS = 'github-creds'  // Jenkins credential ID for GitHub (if repository is private)
    }

    stages {
        stage('Install Docker') {
            steps {
                script {
                    // Install Docker if it's not already installed
                    if (sh(script: 'which docker', returnStatus: true) != 0) {
                        echo "Docker not found. Installing Docker..."
                        sh """
                            sudo apt-get update
                            sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
                            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                            sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
                            sudo apt-get update
                            sudo apt-get install -y docker-ce
                            sudo systemctl start docker
                            sudo systemctl enable docker
                        """
                    } else {
                        echo "Docker is already installed."
                    }
                }
            }
        }

        stage('Clone GitHub Repo') {
            steps {
                script {
                    // Clone the GitHub repository with credentials if private
                    git credentialsId: GITHUB_CREDENTIALS, url: GITHUB_REPO_URL
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repository
                    docker.build(DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin"
                        sh "docker push $DOCKER_IMAGE_NAME"
                    }
                }
            }
        }

        // Optional: You can add a step to create and run a container from the pushed image
        stage('Run Docker Container') {
            steps {
                script {
                    // Pull the image from Docker Hub and run the container (optional step)
                    sh "docker pull $DOCKER_IMAGE_NAME"
                    sh "docker run -d -p 5000:5000 $DOCKER_IMAGE_NAME"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-flask-app1'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                }
            }
        }

        stage('Push Docker Image to Docker Hub some changes') {
            steps {
                script {
                   echo 'pushing to Docker Hub...'
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

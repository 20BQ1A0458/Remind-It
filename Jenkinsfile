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
                    withCredentials([usernamePassword(credentialsId: 'docker-c', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        def dockerImageWithRepo = "${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        def buildStatus = bat(script: "docker build -t ${dockerImageWithRepo} .", returnStatus: true)
                        if (buildStatus != 0) {
                            error 'Docker build failed!'
                        }
                        echo "Docker image built: ${dockerImageWithRepo}"
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-c', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo 'Logging in to Docker Hub...'
                        def loginStatus = bat(script: """
                            docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%
                        """, returnStatus: true)
                        if (loginStatus != 0) {
                            error 'Docker login failed!'
                        }

                        def dockerImageWithRepo = "${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        echo "Pushing Docker image: ${dockerImageWithRepo}"
                        def pushStatus = bat(script: "docker push ${dockerImageWithRepo}", returnStatus: true)
                        if (pushStatus != 0) {
                            error 'Docker push failed!'
                        }
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

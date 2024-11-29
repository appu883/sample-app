pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "appu883/sample-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-token"
        SSH_CREDENTIALS_ID = "ssh_key"
        SERVER_2_USER = "ubuntu"
        SERVER_2_IP = "172.31.5.201"
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo "Checking out the repository..."
                git branch: 'main', url: 'https://github.com/appu883/sample-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building the Docker image..."
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE}:v${BUILD_ID} . 
                    docker tag ${DOCKER_IMAGE}:v${BUILD_ID} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}", url: 'https://index.docker.io/v1/']) {
                    sh """
                    docker push ${DOCKER_IMAGE}:v${BUILD_ID}
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Cleanup Existing Container on Server 1') {
            steps {
                echo "Stopping and removing any existing container on Server 1..."
                sh """
                docker stop sample-app || true
                docker rm sample-app || true
                """
            }
        }

        stage('Run Container on Server 1') {
            steps {
                echo "Running the Docker container on Server 1..."
                sh """
                docker run -itd --name sample-app -p 3001:3001 ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to Server 2') {
            steps {
                echo "Deploying the Docker image to Server 2..."

                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${SERVER_2_USER}@${SERVER_2_IP} '
                        docker pull ${DOCKER_IMAGE}:latest && 
                        docker stop sample-app || true &&
                        docker rm sample-app || true &&
                        docker run -d --name sample-app -p 3001:3001 ${DOCKER_IMAGE}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. The container is running, and the image has been pushed to Docker Hub."
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
        always {
            echo "Cleaning up unused Docker images..."
            sh "docker image prune -f"
        }
    }
}

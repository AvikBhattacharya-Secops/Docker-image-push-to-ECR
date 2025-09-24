pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '439110395780'
        ECR_REPOSITORY = 'my-repo'

        // Docker Hub details
        DOCKER_HUB_USERNAME = 'avikbhattacharya056'
        DOCKER_HUB_REPOSITORY = 'my-second-image'

        // Common image name and tag
        IMAGE_NAME = "${DOCKER_HUB_REPOSITORY}"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Tag Docker Image for ECR') {
            steps {
                script {
                    sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh """
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USERNAME_CRED', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    script {
                        sh """
                        echo ${DOCKER_HUB_PASSWORD} | docker login --username ${DOCKER_HUB_USERNAME_CRED} --password-stdin
                        """
                    }
                }
            }
        }

        stage('Tag Docker Image for Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USERNAME_CRED', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    script {
                        sh """
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USERNAME_CRED}/${DOCKER_HUB_REPOSITORY}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USERNAME_CRED', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    script {
                        sh """
                        docker push ${DOCKER_HUB_USERNAME_CRED}/${DOCKER_HUB_REPOSITORY}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Cleanup Docker Images (Optional)') {
            steps {
                script {
                    sh """
                    docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                    docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG} || true
                    docker rmi ${DOCKER_HUB_USERNAME}/${DOCKER_HUB_REPOSITORY}:${IMAGE_TAG} || true
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image successfully pushed to both AWS ECR and Docker Hub!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}

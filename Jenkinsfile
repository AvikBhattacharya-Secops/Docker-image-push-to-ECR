pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'my-docker-repo'
        IMAGE_TAG = 'latest'
        AWS_CREDENTIALS_ID = 'aws-ecr-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def accountId = sh(
                        script: "aws sts get-caller-identity --query Account --output text",
                        returnStdout: true
                    ).trim()

                    def ecrUri = "${accountId}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}"

                    sh "docker build -t ${ecrUri}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Authenticate with ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${env.AWS_CREDENTIALS_ID}"]]) {
                    script {
                        def accountId = sh(
                            script: "aws sts get-caller-identity --query Account --output text",
                            returnStdout: true
                        ).trim()

                        def ecrUri = "${accountId}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"

                        sh """
                            aws --version
                            aws ecr get-login-password --region ${env.AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ecrUri}
                        """
                    }
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    def accountId = sh(
                        script: "aws sts get-caller-identity --query Account --output text",
                        returnStdout: true
                    ).trim()

                    def ecrUri = "${accountId}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}"

                    sh "docker push ${ecrUri}:${env.IMAGE_TAG}"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Docker image built and pushed to ECR successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the logs for details.'
        }
    }
}

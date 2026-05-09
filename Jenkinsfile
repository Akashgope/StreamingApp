pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ACCOUNT_ID = '485844165762'
        ECR_REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    sh "docker build -t streamingapp-frontend ./frontend"
                    sh "docker tag streamingapp-frontend:latest ${ECR_REGISTRY}/streamingapp-frontend:latest"
                    sh "docker push ${ECR_REGISTRY}/streamingapp-frontend:latest"
                    
                    def services = ['authService', 'streamingService', 'adminService', 'chatService']
                    for (service in services) {
                        sh "docker build -t streamingapp-${service} -f backend/Dockerfile ./backend --build-arg SERVICE_NAME=${service}"
                        sh "docker tag streamingapp-${service}:latest ${ECR_REGISTRY}/streamingapp-backend-${service}:latest"
                        sh "docker push ${ECR_REGISTRY}/streamingapp-backend-${service}:latest"
                    }
                }
            }
        }
    }
}


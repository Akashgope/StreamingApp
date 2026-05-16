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
                    
                    def services = [
                        [name: 'authService', repo: 'auth'],
                        [name: 'streamingService', repo: 'streaming'],
                        [name: 'adminService', repo: 'admin'],
                        [name: 'chatService', repo: 'chat']
                    ]
                    for (service in services) {
                        sh "docker build -t streamingapp-${service.repo} -f backend/Dockerfile ./backend --build-arg SERVICE_NAME=${service.name}"
                        sh "docker tag streamingapp-${service.repo}:latest ${ECR_REGISTRY}/streamingapp-backend-${service.repo}:latest"
                        sh "docker push ${ECR_REGISTRY}/streamingapp-backend-${service.repo}:latest"
                    }
                }
            }
        }
    }
}


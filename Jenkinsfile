pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        ECR_REPO = "048119078172.dkr.ecr.us-east-1.amazonaws.com/myapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/disisdsid/sample-node-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ECR_REPO%:build-%BUILD_NUMBER% ."
            }
        }

        stage('Login to ECR') {
            steps {
                bat "aws ecr get-login-password --region %AWS_DEFAULT_REGION% | docker login --username AWS --password-stdin %ECR_REPO%"
            }
        }

        stage('Push to ECR') {
            steps {
                bat "docker push %ECR_REPO%:build-%BUILD_NUMBER%"
            }
        }

        stage('Deploy to ECS') {
            steps {
                bat "aws ecs update-service --cluster myCluster --service myService --force-new-deployment --region %AWS_DEFAULT_REGION%"
            }
        }
    }
}

pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        ECR_REPO = "048119078172.dkr.ecr.ap-south-1.amazonaws.com/myapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/disisdsid/sample-node-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ECR_REPO%:build-%BUILD_NUMBER% ."
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-creds']]) {
                    bat "aws ecr get-login-password --region %AWS_DEFAULT_REGION% | docker login --username AWS --password-stdin %ECR_REPO%"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                bat "docker push %ECR_REPO%:build-%BUILD_NUMBER%"
            }
        }

        stage('Register Task Definition & Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-creds']]) {
                    bat """
                    copy taskdef.json taskdef-updated.json
                    powershell -Command "(Get-Content taskdef-updated.json) -replace 'IMAGE_TAG', 'build-%BUILD_NUMBER%' | Set-Content taskdef-updated.json"
                    aws ecs register-task-definition --cli-input-json file://taskdef-updated.json
                    aws ecs update-service --cluster myCluster-dev --service myService-dev --task-definition myTaskDef-dev --region %AWS_DEFAULT_REGION%
                    """
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "048119078172.dkr.ecr.us-east-1.amazonaws.com/myapp"
        IMAGE_TAG = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service \
                  --cluster myCluster \
                  --service myService \
                  --force-new-deployment \
                  --region $AWS_REGION
                """
            }
        }
    }
}

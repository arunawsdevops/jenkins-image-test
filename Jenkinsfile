pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION    = 'your-region' // Replace with your AWS region
        AWS_ACCOUNT_ID        = '670004487191' // Replace with your AWS account ID
        ECR_REPO_NAME         = 'test-project-repo'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Push to ECR') {
            steps {
                withCredentials([string(credentialsId: 'aws-cred', variable: 'AWS_CREDENTIALS')]) {
                    script {
                        // AWS CLI login to ECR
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        
                        // Build Docker image
                        sh "docker build -t ${ECR_REPO_NAME} ."
                        
                        // Tag the image
                        sh "docker tag ${ECR_REPO_NAME}:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                        
                        // Push image to ECR
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to ECS') {
            steps {
                script {
                    // AWS CLI update ECS service
                    sh "aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service ${ECS_SERVICE_NAME} --force-new-deployment --region ${AWS_DEFAULT_REGION}"
                }
            }
        }
    }
}

pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION    = 'us-east-1' // Replace with your AWS region
        AWS_ACCOUNT_ID        = '670004487191' // Replace with your AWS account ID
        ECR_REPO_NAME         = 'test-project-repo'
        ECS_CLUSTER_NAME      = 'green-app-cluster'
        ECS_SERVICE_NAME      = 'green-app-service'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                // Checkout code from Git
                checkout([$class: 'GitSCM', branches: [[name: '*/*']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-git', url: 'https://github.com/arunawsdevops/Car-web.git']]])
                sh "ls -l"
            }
        }
        
        stage('Build and Push to ECR') {
            steps {
                script {
                    // Login to ECR
                    withAWS(credentials: 'aws-cred', region: AWS_DEFAULT_REGION) {
                        // Get Docker login command and execute
                        def ecrLogin = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                        sh "echo \${ecrLogin} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        
                        // Build Docker image
                        sh "docker build -t ${ECR_REPO_NAME} . -f Dockerfile"
                        
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
                    // Update ECS service with new image
                    withAWS(credentials: 'aws-cred', region: AWS_DEFAULT_REGION) {
                        sh "aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service ${ECS_SERVICE_NAME} --force-new-deployment --region ${AWS_DEFAULT_REGION}"
                    }
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "027275363480.dkr.ecr.us-east-1.amazonaws.com/backend-service"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/LakshminarayanaTejas/backend-server.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                    sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t backend-service ."
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh "docker tag backend-service:latest $ECR_REPO:latest"
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push $ECR_REPO:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@107.21.180.17 '
                        docker pull $ECR_REPO:latest &&
                        docker stop backend || true &&
                        docker rm backend || true &&
                        docker run -d -p 5000:5000 --name backend $ECR_REPO:latest
                    '
                    """
                }
            }
        }
    }
}

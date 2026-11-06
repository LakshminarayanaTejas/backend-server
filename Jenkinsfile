pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "027275363480.dkr.ecr.us-east-1.amazonaws.com/backend-service"
    }

    stages {
        stage('Checkout from Git') {
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-credentials']]) {
                    sshagent(credentials: ['ec2-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@54.90.100.31 '
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO &&
                            docker pull $ECR_REPO:latest &&
                            docker stop backend || true &&
                            docker rm -f backend || true &&
                            docker ps -q --filter expose=5000 | xargs -r docker stop &&
                            docker ps -a -q --filter expose=5000 | xargs -r docker rm &&
                            docker run -d -p 5000:5000 --name backend $ECR_REPO:latest
                        '
                        """
                    }
                }
            }
        }
    }
}


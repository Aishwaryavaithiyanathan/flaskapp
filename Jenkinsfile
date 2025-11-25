pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/flask-devops"
        EC2_USER = "ec2-user"
        EC2_HOST = "<EC2_PUBLIC_IP>"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-user>/<your-repo>.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                        docker pull ${IMAGE_NAME}:latest &&
                        docker stop flask-app || true &&
                        docker rm flask-app || true &&
                        docker run -d --name flask-app -p 5000:5000 ${IMAGE_NAME}:latest
                        '
                    """
                }
            }
        }
    }

    post {
        success { echo "🚀 Container deployment successful!" }
        failure { echo "❌ Deployment failed!" }
    }
}

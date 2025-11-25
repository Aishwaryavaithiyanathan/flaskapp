pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // DockerHub username/password credential ID in Jenkins
        EC2_CREDENTIAL_ID = 'ec2-ssh-key'           // Jenkins SSH credential ID for EC2
        EC2_USER = 'ec2-user'
        EC2_IP = '98.92.82.8'                       // Replace with your EC2 public IP
        IMAGE_NAME = 'aishwaryavaithiyanathan/flaskapp'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git(
                    url: 'https://github.com/Aishwaryavaithiyanathan/flaskapp',
                    branch: 'main',
                    credentialsId: 'github-token'     // GitHub personal access token credential ID in Jenkins
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    bat "docker login -u %USERNAME% -p %PASSWORD%"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([EC2_CREDENTIAL_ID]) {
                    bat """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} ^
                    "docker pull ${IMAGE_NAME}:latest && ^
                    docker stop flaskapp || true && ^
                    docker rm flaskapp || true && ^
                    docker run -d --name flaskapp -p 5000:5000 ${IMAGE_NAME}:latest"
                    """
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}

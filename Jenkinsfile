pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'docker-hub-creds'    // Jenkins credential ID for DockerHub
        EC2_SSH_KEY = 'Jenkinskp'               // SSH key credential ID
        EC2_USER = "ec2-user"
        EC2_IP = "98.92.82.8"                 // replace with EC2 IP
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/aishwaryavaithiyanathan/flask-app.git'
                credentialsId: 'github-token'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "docker login -u $USERNAME -p $PASSWORD"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([EC2_SSH_KEY]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        docker stop flask-container || true
                        docker rm flask-container || true
                        docker pull ${IMAGE_NAME}:latest
                        docker run -d -p 5000:5000 --name flask-container ${IMAGE_NAME}:latest
                    '
                    """
                }
            }
        }
    }
}

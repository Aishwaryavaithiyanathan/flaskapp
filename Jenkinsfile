pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'    // DockerHub creds in Jenkins
        IMAGE_NAME = 'yourdockerhubusername/flask-app'
        EC2_USER = 'ec2-user'
        EC2_HOST = 'YOUR_EC2_PUBLIC_IP'
        SSH_KEY = 'ec2-private-key'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/yourrepo/flask-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent([SSH_KEY]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            sudo docker pull ${IMAGE_NAME}:latest;
                            sudo docker rm -f flaskapp || true;
                            sudo docker run -d --name flaskapp -p 80:5000 ${IMAGE_NAME}:latest
                        '
                        """
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Deployment Completed Successfully!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}

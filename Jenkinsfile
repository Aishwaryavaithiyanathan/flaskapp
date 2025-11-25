pipeline {
    agent any

    environment {
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
        IMAGE_TAG = "latest"
        DOCKERHUB_CREDENTIALS = "dockerhub_creds" // Make sure this matches your Jenkins credentials ID
        EC2_USER = "ec2-user"   // Replace with your EC2 username
        EC2_HOST = "98.92.82.8"  // Replace with your EC2 public IP
        SSH_KEY = "ec2_ssh_key"  // Jenkins credential ID for your private SSH key
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
                git branch: 'main', url: 'https://github.com/Aishwaryavaithiyanathan/flaskapp'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.push("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${SSH_KEY}", 
                                                  keyFileVariable: 'SSH_KEY_PATH', 
                                                  usernameVariable: 'SSH_USER')]) {
                    sh """
                    ssh -i $SSH_KEY_PATH -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        docker pull ${IMAGE_NAME}:${IMAGE_TAG} &&
                        docker stop flaskapp || true &&
                        docker rm flaskapp || true &&
                        docker run -d --name flaskapp -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

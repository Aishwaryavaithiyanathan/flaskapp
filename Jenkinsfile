pipeline {
    agent any

    environment {
        DOCKER_HUB_CRED = credentials('dockerhub-creds') // Jenkins credential ID
        EC2_USER = 'ec2-user'                           // EC2 username
        EC2_HOST = '3.92.91.238'                      // EC2 public IP
        SSH_KEY = credentials('ec2-ssh-key')            // SSH private key stored in Jenkins
        IMAGE_NAME = 'aishwaryavaithiyanathan/flaskapp:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Aishwaryavaithiyanathan/flaskapp.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    bat """
ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} "docker pull ${IMAGE_NAME} && docker stop flaskapp || true && docker rm flaskapp || true && docker run -d --name flaskapp -p 5000:5000 ${IMAGE_NAME}"
"""

                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to EC2 successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}

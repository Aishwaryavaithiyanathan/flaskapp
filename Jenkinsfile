pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub_creds' // Docker Hub username/password
        EC2_KEY = 'ec2-ssh-key' // EC2 private key
        EC2_USER = "ec2-user"
        EC2_HOST = "98.92.82.8" // replace with your EC2 IP
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp:latest"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git(
                    url: 'https://github.com/Aishwaryavaithiyanathan/flaskapp',
                    branch: 'main',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                bat """
                echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Use direct SSH instead of ssh-agent
                bat """
                ssh -i ${EC2_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} ^
                "docker pull ${IMAGE_NAME} && docker stop flaskapp || true && docker rm flaskapp || true && docker run -d --name flaskapp -p 5000:5000 ${IMAGE_NAME}"
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

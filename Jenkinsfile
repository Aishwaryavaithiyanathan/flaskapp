pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'    // Jenkins credential ID for DockerHub
        EC2_SSH_KEY = 'Jenkinskp'               // SSH key credential ID
        EC2_USER = "ec2-user"
        EC2_IP = "98.92.82.8"                 // replace with EC2 IP
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
    }

    stages {

        stage('Checkout Code') {
            steps {
               git(
    url: 'https://github.com/aishwaryavaithiyanathan/flaskapp',
    branch: 'main',
    credentialsId: 'github-token'
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
                    bat "docker login -u $USERNAME -p $PASSWORD"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                bat "docker push ${IMAGE_NAME}:latest"
            }
        }
stage('Deploy to EC2') {
    bat """
    echo Running deployment on EC2
    ssh -i C:\\Users\\Administrator\\Downloads\\ec2-key.pem -o StrictHostKeyChecking=no ec2-user@<EC2-PUBLIC-IP> "docker pull aishwaryavaithiyanathan/flaskapp:latest && docker stop flaskapp || true && docker rm flaskapp || true && docker run -d --name flaskapp -p 5000:5000 aishwaryavaithiyanathan/flaskapp:latest"
    """
}

        
            }
        }
    }
}

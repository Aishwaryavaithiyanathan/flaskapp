pipeline {
    agent any

    environment {
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
        BUILD_TAG = "latest"
        GIT_URL = "https://github.com/Aishwaryavaithiyanathan/flaskapp.git"
        GIT_BRANCH = "main"
        EC2_HOST = "98.92.82.8" // Replace with your EC2 IP
        EC2_USER = "ec2-user"   // Adjust if needed
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Explicit Git checkout with GitHub credentials
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}", credentialsId: 'github-token'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    bat "docker build -t ${IMAGE_NAME}:${BUILD_TAG} ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    // Login using DockerHub credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat "docker push ${IMAGE_NAME}:${BUILD_TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'PEM_FILE', usernameVariable: 'SSH_USER')]) {

                        // Fix Windows SSH key permissions
                        bat """
                        icacls "%PEM_FILE%" /inheritance:r
                        icacls "%PEM_FILE%" /grant:r "%USERNAME%:R"
                        """

                        // SSH and deploy container
                        bat """
                        ssh -i %PEM_FILE% -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} ^
                        "docker pull ${IMAGE_NAME}:${BUILD_TAG} ^
                         && docker stop flaskapp || true ^
                         && docker rm flaskapp || true ^
                         && docker run -d --name flaskapp -p 5000:5000 ${IMAGE_NAME}:${BUILD_TAG}"
                        """
                    }
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

pipeline {
    agent any

    environment {
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
        IMAGE_TAG  = "latest"
        GIT_URL    = "https://github.com/Aishwaryavaithiyanathan/flaskapp.git"
        GIT_BRANCH = "main"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}", credentialsId: 'github-token'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Ensure Minikube docker-env is used
                    bat "minikube docker-env --shell=cmd > minikube-env.cmd"
                    bat "call minikube-env.cmd"

                    // Apply Kubernetes manifests
                    bat "kubectl apply -f k8s/deployment.yml"
                    bat "kubectl apply -f k8s/service.yml"
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

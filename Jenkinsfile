pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'        
        EC2_SSH_KEY = 'ec2-ssh-key'                
        EC2_IP = "98.92.82.8"                      
        IMAGE_NAME = "aishwaryavaithiyanathan/flaskapp"
        GIT_CREDENTIALS = 'github-token'           // GitHub credential ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                git(
                    url: 'https://github.com/aishwaryavaithiyanathan/flaskapp',
                    branch: 'main',
                    credentialsId: "${GIT_CREDENTIALS}"
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

        stage('Push Image to DockerHub') {
            steps {
                bat "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${EC2_SSH_KEY}", keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    bat """
                      echo Deploying container to EC2...

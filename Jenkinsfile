pipeline {
    agent any
    
    environment {
        PATH = "/var/jenkins_home/bin:$PATH"
        DOCKER_IMAGE = "umair1987/todo-backend"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NILEG/devops-backend.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/backend backend=${DOCKER_IMAGE}:${DOCKER_TAG} -n todo-app
                        kubectl rollout status deployment/backend -n todo-app
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Backend deployment successful!'
        }
        failure {
            echo 'Backend deployment failed!'
        }
    }
}
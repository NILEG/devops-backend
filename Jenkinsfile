pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:3345.v03dee9b_f88fc-6
    args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
  - name: docker-tools
    image: docker:27.2.0-cli
    command: ['cat']
    tty: true
    volumeMounts:
      - mountPath: /var/run/docker.sock
        name: docker-sock
  # NEW: Add this container for kubectl
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }
    
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
                container('docker-tools') {
                    sh 'docker build -t umair1987/todo-backend:1 .'
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                container('docker-tools') {
                    script {
                        // Replace 'docker-credentials-id' with the actual ID from your Jenkins credentials
                        withDockerRegistry(credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/') {
                            sh 'docker push umair1987/todo-backend:1'
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    // Update the image version to match what you pushed (e.g., :1)
                    sh 'kubectl set image deployment/backend backend=umair1987/todo-backend:1 -n todo-app'
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
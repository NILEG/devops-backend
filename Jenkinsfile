pipeline {
  environment {
    // REPLACE THIS with your actual DockerHub username
    dockerimagename = "umair1987/express-backend"
    dockerImage = ""
  }
  agent any
  stages {
    
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image') {
      environment {
         // Ensure you have created this ID in Jenkins Credentials
         registryCredential = 'dockerhub-credentials'
      }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
    stage('Deploying to Kubernetes') {
      steps {
        script {
          // Deploy both the deployment and service files
          kubernetesDeploy(configs: 'deployment.yaml, service.yaml')
        }
      }
    }
  }
}
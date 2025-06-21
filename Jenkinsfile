pipeline {
  agent any
  environment {
    // Configure these in Jenkins credentials
    DOCKER_REGISTRY = 'docker.io'
    IMAGE_NAME      = "cloudfire2025/cloudfire"
    KUBE_CONFIG_CRED = 'k8s'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.withRegistry("https://${DOCKER_REGISTRY}", 'dockerhub') {
            def img = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
            img.push()
            img.push('latest')
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: KUBE_CONFIG_CRED, variable: 'KUBECONFIG')]) {
          sh "kubectl apply -f k8s/deployment.yaml"
          sh "kubectl apply -f k8s/service.yaml"
        }
      }
    }
  }
}
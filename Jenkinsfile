pipeline {
  agent any
  environment {
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
          docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
        }
      }
    }
    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASSWORD')]) {
          bat "docker login -u %USER% -p %PASSWORD% docker.io"
        }
      }
    }
    stage('Push image') {
      steps {
        bat "docker push ${IMAGE_NAME}:${env.BUILD_ID}"
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: KUBE_CONFIG_CRED, variable: 'KUBECONFIG')]) {
          bat "kubectl apply -f k8s\\deployment.yaml"
          bat "kubectl apply -f k8s\\service.yaml"
        }
      }
    }
  }
}
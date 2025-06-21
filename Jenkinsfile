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
          sh 'echo $PASSWORD | docker login -u $USER --password-stdin docker.io'
        }
      }
    }
    stage('Push image') {
      steps {
        sh "docker push ${IMAGE_NAME}:${env.BUILD_ID}"
      }
    }
    stage('Kubernetes Deploy') {
      steps {
        withCredentials([file(credentialsId: 'k8s', variable: 'KUBECONFIG')]) {
          sh '''
            kubectl get nodes
            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service.yaml
          '''
        }
      }
    }
  }
}
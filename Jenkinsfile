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
    stage('Check K8s Connection (AWS)') {
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
          ]
        ]) {
          sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config
            kubectl get pods
          '''
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
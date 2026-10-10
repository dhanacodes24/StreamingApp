pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'
    ECR_REGISTRY = '994114819227.dkr.ecr.us-east-1.amazonaws.com'
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Login to ECR') {
      steps {
        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY'
      }
    }
    stage('Build & Push Images') {
      steps {
        sh 'docker build -t $ECR_REGISTRY/streaming-auth:1.0.$BUILD_NUMBER backend/authService'
        sh 'docker push $ECR_REGISTRY/streaming-auth:1.0.$BUILD_NUMBER'
        // repeat build+push for streaming-stream, streaming-admin, streaming-chat, streaming-frontend
      }
    }
  }
}


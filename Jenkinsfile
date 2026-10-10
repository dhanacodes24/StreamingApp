pipeline {
  agent any
  options {
    timestamps()   // ✅ valid option
  }
  environment {
    AWS_REGION   = 'us-east-1'
    ECR_REGISTRY = '994114819227.dkr.ecr.us-east-1.amazonaws.com'
  }
  stages {
    stage('Checkout') {
      steps {
        echo "📥 Starting SCM checkout..."
        checkout scm
        echo "✅ Checkout complete."
      }
    }
    stage('Login to ECR') {
      steps {
        echo "🔑 Logging in to AWS ECR..."
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'dhana_aws_creds']]) {
          sh '''
            set -x
            aws ecr get-login-password --region $AWS_REGION \
              | docker login --username AWS --password-stdin $ECR_REGISTRY
          '''
        }
        echo "✅ Login to ECR succeeded."
      }
    }
    stage('Build & Push Images') {
      steps {
        echo "�� Building and pushing streaming-auth image..."
        sh '''
          set -x
          docker build -t $ECR_REGISTRY/streaming-auth:1.0.$BUILD_NUMBER backend/authService
          docker push $ECR_REGISTRY/streaming-auth:1.0.$BUILD_NUMBER
        '''
        echo "✅ streaming-auth image pushed successfully."

        echo "🚀 Building and pushing streaming-admin image..."
        sh '''
          set -x
          docker build -t $ECR_REGISTRY/streaming-admin:1.0.$BUILD_NUMBER backend/adminService
          docker push $ECR_REGISTRY/streaming-admin:1.0.$BUILD_NUMBER
        '''
        echo "✅ streaming-admin image pushed successfully."

        // repeat for streaming-stream, streaming-chat, streaming-frontend
      }
    }
  }
}

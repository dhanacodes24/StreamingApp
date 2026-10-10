pipeline {
  agent any

  environment {
    AWS_REGION   = 'us-east-1'
    ECR_REGISTRY = '994114819227.dkr.ecr.us-east-1.amazonaws.com'
    IMAGE_TAG    = "1.0.${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Login to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'dhana_aws_creds']]) {
          sh '''
            aws ecr get-login-password --region $AWS_REGION | \
              docker login --username AWS --password-stdin $ECR_REGISTRY
          '''
        }
      }
    }

    stage('Build Images') {
      steps {
        sh '''
          set -e
          docker build -t $ECR_REGISTRY/streaming-auth:$IMAGE_TAG backend/authService

          docker build -t $ECR_REGISTRY/streaming-stream:$IMAGE_TAG \
            -f backend/streamingService/Dockerfile backend

          docker build -t $ECR_REGISTRY/streaming-admin:$IMAGE_TAG \
            -f backend/adminService/Dockerfile backend

          docker build -t $ECR_REGISTRY/streaming-chat:$IMAGE_TAG \
            -f backend/chatService/Dockerfile backend

          docker build -t $ECR_REGISTRY/streaming-frontend:$IMAGE_TAG frontend
        '''
      }
    }

    stage('Push Images') {
      steps {
        sh '''
          set -e
          for s in auth stream admin chat frontend; do
            echo "Pushing streaming-$s:$IMAGE_TAG"
            docker push $ECR_REGISTRY/streaming-$s:$IMAGE_TAG
          done
        '''
      }
    }
  }

  post {
    success {
      echo "All 5 images pushed with tag ${IMAGE_TAG}"
    }
    failure {
      echo "Build failed. Check the stage that turned red."
    }
    always {
      sh 'docker logout $ECR_REGISTRY || true'
    }
  }
}

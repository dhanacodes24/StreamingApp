// Helper: publish a Chatbot-formatted message to an SNS topic
def notifySlack(String topicArn, String title, String details) {
  withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'dhana_aws_creds']]) {
    withEnv(["TOPIC_ARN=${topicArn}", "MSG_TITLE=${title}", "MSG_BODY=${details}"]) {
      sh '''
        MSG=$(printf '{"version":"1.0","source":"custom","content":{"textType":"client-markdown","title":"%s","description":"%s"}}' "$MSG_TITLE" "$MSG_BODY")
        aws sns publish --region $AWS_REGION --topic-arn "$TOPIC_ARN" --message "$MSG" \
          || echo "WARNING: SNS notification failed (build result not affected)"
      '''
    }
  }
}

pipeline {
  agent any

  environment {
    AWS_REGION        = 'us-east-1'
    ECR_REGISTRY      = '994114819227.dkr.ecr.us-east-1.amazonaws.com'
    IMAGE_TAG         = "1.0.${BUILD_NUMBER}"
    SNS_SUCCESS_TOPIC = 'arn:aws:sns:us-east-1:994114819227:streamingapp-deploy-success'
    SNS_FAILURE_TOPIC = 'arn:aws:sns:us-east-1:994114819227:streamingapp-deploy-failure'
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

          docker build -t $ECR_REGISTRY/streaming-frontend:$IMAGE_TAG \
            --build-arg REACT_APP_AUTH_API_URL=/api/auth/api \
            --build-arg REACT_APP_STREAMING_API_URL=/api/streaming/api \
            --build-arg REACT_APP_STREAMING_PUBLIC_URL=/api/streaming \
            --build-arg REACT_APP_ADMIN_API_URL=/api/admin/api/admin \
            --build-arg REACT_APP_CHAT_API_URL=/api/chat/api/chat \
            --build-arg REACT_APP_CHAT_SOCKET_URL=http://streamingapp.local \
            frontend
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
      notifySlack(env.SNS_SUCCESS_TOPIC,
                  "Build succeeded",
                  "${env.JOB_NAME} #${env.BUILD_NUMBER} - images tagged ${env.IMAGE_TAG} pushed to ECR - ${env.BUILD_URL}")
    }
    failure {
      echo "Build failed. Check the stage that turned red."
      notifySlack(env.SNS_FAILURE_TOPIC,
                  "Build FAILED",
                  "${env.JOB_NAME} #${env.BUILD_NUMBER} failed - ${env.BUILD_URL}")
    }
    always {
      sh 'docker logout $ECR_REGISTRY || true'
    }
  }
}

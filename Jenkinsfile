pipeline {
  agent any

  environment {
    // AWS
    AWS_REGION     = 'ap-south-1'
    AWS_ACCOUNT_ID = '218173910089'

    // App
    APP_NAME   = 'java-app'
    IMAGE_TAG  = "${BUILD_NUMBER}"

    // ECR (derived variable)
    ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"
  }

  stages {

    stage('Build & Test') {
      steps {
        sh 'mvn clean test'
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          docker build \
            -t $ECR_REPO:$IMAGE_TAG \
            -t $ECR_REPO:latest .
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region $AWS_REGION |
          docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

          docker push $ECR_REPO:$IMAGE_TAG
          docker push $ECR_REPO:latest
        '''
      }
    }
  }
}

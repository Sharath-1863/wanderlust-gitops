pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'sharath2003'
    BACKEND_IMAGE  = "${DOCKERHUB_USER}/wanderlust-backend"
    FRONTEND_IMAGE = "${DOCKERHUB_USER}/wanderlust-frontend"
    IMAGE_TAG      = "${BUILD_NUMBER}"
    API_PATH       = 'http://localhost:5000'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build backend image') {
      steps { sh 'docker build -t $BACKEND_IMAGE:$IMAGE_TAG backend' }
    }

    stage('Build frontend image') {
      steps { sh 'docker build -t $FRONTEND_IMAGE:$IMAGE_TAG --build-arg VITE_API_PATH=$API_PATH frontend' }
    }

    stage('Push images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $BACKEND_IMAGE:$IMAGE_TAG
            docker push $FRONTEND_IMAGE:$IMAGE_TAG
            docker logout
          '''
        }
      }
    }
  }

  post {
    always { sh 'docker image prune -f' }
  }
}

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
        withCredentials([usernamePassword(credentialsId: 'docker_cred', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $BACKEND_IMAGE:$IMAGE_TAG
            docker push $FRONTEND_IMAGE:$IMAGE_TAG
            docker logout
          '''
        }
      }
    }

    stage('Update manifests') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'github_cred', usernameVariable: 'GH_USER', passwordVariable: 'GH_PASS')]) {
          sh '''
            sed -i "s|image: .*wanderlust-backend:.*|image: $BACKEND_IMAGE:$IMAGE_TAG|" kubernetes/backend.yaml
            sed -i "s|image: .*wanderlust-frontend:.*|image: $FRONTEND_IMAGE:$IMAGE_TAG|" kubernetes/frontend.yaml
            git config user.email "jenkins@local"
            git config user.name "jenkins"
            git add kubernetes/
            git commit -m "ci: update images to $IMAGE_TAG" || exit 0
            git push https://$GH_USER:$GH_PASS@github.com/Sharath-1863/wanderlust-gitops.git HEAD:main
          '''
        }
      }
    }
  }

  post {
    always { sh 'docker image prune -f' }
  }
}
// .....existing code.....
pipeline {
  agent any
  environment {
    FRONTEND_DIR = 'frontend'
    BACKEND_DIR  = 'backend'
    REG_USER_ID  = 'ghcr-creds'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Frontend') {
      steps {
        dir("${FRONTEND_DIR}") {
          sh 'docker build -t portfolio-frontend:${GIT_COMMIT} .'
        }
      }
    }

    stage('Build Backend') {
      steps {
        dir("${BACKEND_DIR}") {
          sh 'docker build -t portfolio-backend:${GIT_COMMIT} .'
        }
      }
    }

    stage('Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.REG_USER_ID, usernameVariable: 'REG_USER', passwordVariable: 'REG_PW')]) {
          sh """#!/bin/bash
            set -e
            echo "$REG_PW" | docker login ghcr.io -u "$REG_USER" --password-stdin
            docker tag portfolio-frontend:${GIT_COMMIT} ghcr.io/${REG_USER}/devops-portfolio-frontend:${GIT_COMMIT}
            docker tag portfolio-backend:${GIT_COMMIT}  ghcr.io/${REG_USER}/devops-portfolio-backend:${GIT_COMMIT}
            docker push ghcr.io/${REG_USER}/devops-portfolio-frontend:${GIT_COMMIT}
            docker push ghcr.io/${REG_USER}/devops-portfolio-backend:${GIT_COMMIT}
          """
        }
      }
    }

    stage('Save tags') {
      steps {
        sh '''#!/bin/bash
          printf "frontend=ghcr.io/%s/devops-portfolio-frontend:%s\nbackend=ghcr.io/%s/devops-portfolio-backend:%s\n" "$REG_USER" "$GIT_COMMIT" "$REG_USER" "$GIT_COMMIT" > image-tags.txt
        '''
        archiveArtifacts artifacts: 'image-tags.txt'
      }
    }
  }

  post {
    success {
      echo 'Pipeline succeeded'
    }
    failure {
      echo 'Pipeline failed'
    }
  }
}
// .....existing code.....
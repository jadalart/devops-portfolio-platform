pipeline {
    agent any
    environment {
      FRONTEND_DIR = 'frontend'
      BACKEND_DIR  = 'backend'
      REG_USER_ID  = 'ghcr-creds'
    }
    stages {
        stage('checkout'){
            steps { checkout scm }
        }
        stage('build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'docker build -t portfolio-frontend:${GIT_COMMIT} .'
                }
            }
        } 
        stage('build Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'docker build -t portfolio-backend:${GIT_COMMIT} .'
                }
            }
        }
        stage('Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.REG_USER_ID, usernameVariable: 'REG_USER', passwordVariable: 'REG_PW')]) {
                  sh '''
                    echo "$REG_PW" | docker login ghcr.io -u "$REG_USER" --password-stdin
                    docker tag portfolio-frontend:${GIT_COMMIT} ghcr.io/${REG_USER}/devops-portfolio-frontend:${GIT_COMMIT}
                    docker tag portfolio-backend:${GIT_COMMIT}  ghcr.io/${REG_USER}/devops-portfolio-backend:${GIT_COMMIT}
                    docker push ghcr.io/${REG_USER}/devops-portfolio-frontend:${GIT_COMMIT}
                    docker push ghcr.io/${REG_USER}/devops-portfolio-backend:${GIT_COMMIT}
          '''  
                }
            }
    }
    stage('save tags') {
        steps {
          sh 'echo "frontend=ghcr.io/${REG_USER}/devops-portfolio-frontend:${GIT_COMMIT}" > image-tags.txt'
          sh 'echo "backend=ghcr.io/${REG_USER}/devops-portfolio-backend:${GIT_COMMIT}" >> image-tags.txt'
          archiveArtifacts artifacts: 'image-tags.txt'
        }
    }
}
post { success { echo 'Pipeline succeeded' } failure { echo 'Pipeline failed' } }
}
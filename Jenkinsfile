pipeline {
  agent any

  environment {
    DOCKER_BUILDKIT = '1'
    IMAGE = 'somasundaram2002/somu'   // change if needed
  }

  tools {
    nodejs 'nodejs'
  }

  stages {
    stage('Prep') {
      steps {
        checkout scm
        script {
          def COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          env.TAG = COMMIT_ID
        }
        echo "Commit id: ${env.TAG}"
      }
    }

    stage('Unit tests') {
      steps {
        sh 'npm ci || npm install'
        sh 'npm test || true'
      }
    }

    stage('Docker build & push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                      usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
        }
        sh "docker build -t ${IMAGE}:${TAG} ."
        sh "docker push ${IMAGE}:${TAG}"
      }
    }

    stage('Cleanup (optional)') {
      steps {
        sh 'docker logout || true'
      }
    }
  }

  options {
    timeout(time: 20, unit: 'MINUTES')
    timestamps()
  }
}

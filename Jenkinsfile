pipeline {
  agent any

  environment {
    DOCKER_BUILDKIT = '1'                               // enable BuildKit
    IMAGE = 'somasundaram2002/somu'
  }

  tools {
    nodejs 'nodejs'
  }

  stages {
    stage('Prep') {
      steps {
        checkout scm
        script {
          COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          env.TAG = COMMIT_ID
        }
        echo "Commit id: ${env.TAG}"
      }
    }

    stage('Unit tests') {
      steps {
        sh 'npm ci'                                      // faster, reproducible
        sh 'npm test'
      }
    }

    stage('Docker buildx push') {
      steps {
        sh 'docker buildx create --use || true'          // idempotent
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                          usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
        }
        // optional: registry cache to avoid rebuilding/pushing unchanged layers
        sh """
          docker buildx build \
            --platform linux/amd64 \
            --cache-to type=registry,ref=${IMAGE}:cache,mode=max \
            --cache-from type=registry,ref=${IMAGE}:cache \
            -t ${IMAGE}:${TAG} --push .
        """
      }
    }

    stage('Cleanup (optional)') {
      steps {
        sh 'docker logout || true'
      }
    }
  }

  options {
    timeout(time: 20, unit: 'MINUTES')                  // avoid stuck runs
  }
}

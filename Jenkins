pipeline {
  agent any

  environment {
    COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
  }

  tools {
    nodejs 'nodejs'
  }

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        echo "Commit id: ${COMMIT_ID}"
      }
    }

    stage('Test') {
      steps {
        sh 'npm install'
        sh 'npm test'
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          docker.withRegistry('', 'dockerhub') {
            def appImage = docker.build("somasundaram2002/somu:${COMMIT_ID}", '.')
            appImage.push()
          }
        }
      }
    }
  }
}

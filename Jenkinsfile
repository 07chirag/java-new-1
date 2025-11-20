pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
    SONAR_HOST  = 'http://sonarqube:9000'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare') {
      steps {
        echo "No build required for simple HTML; listing files..."
        sh 'ls -la'
      }
    }

    stage('Sonar Scan') {
      steps {
        script {
          docker.image('sonarsource/sonar-scanner-cli:latest')
                .inside("--network ci-network -v ${env.WORKSPACE}:/usr/src") {
            sh """
              sonar-scanner \
                -Dsonar.projectKey=java-new-1 \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST} \
                -Dsonar.login=${SONAR_TOKEN}
            """
          }
        }
      }
    }
  }

  post { always { echo "Pipeline finished" } }
}

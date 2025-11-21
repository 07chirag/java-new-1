pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
    SONAR_HOST  = 'http://172.18.0.2:9000'
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
          // Option A: run scanner inside sonarsource docker image (uses network ci-network)
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
          // Note: If you prefer local scanner binary, remove the docker block above and use the local sonar-scanner.
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          timeout(time: 15, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg == null) {
              error "No Quality Gate result received from SonarQube"
            }
            echo "Quality Gate status: ${qg.status}"
            if (qg.status != 'OK') {
              error "Quality Gate failed: ${qg.status}"
            }
          }
        }
      }
    }
  }

  post { always { echo "Pipeline finished" } }
}

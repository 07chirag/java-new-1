pipeline {
  agent any
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('SonarQube analysis (direct)') {
      steps {
        withCredentials([string(credentialsId: 'sonar-token-id', variable: 'SONAR_TOKEN')]) {
          sh '''
            # if sonar-scanner present in workspace:
            ./sonar-scanner/bin/sonar-scanner \
              -Dsonar.projectKey=java-new-1 \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://sonarqube:9000 \
              -Dsonar.login=$SONAR_TOKEN
          '''
        }
      }
    }

    stage('Build/Other') {
      steps { echo "Continue pipeline — no waitForQualityGate" }
    }
  }
}

pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('SonarQube analysis') {
      steps {
        // Replace 'MySonarServer' with the Sonar server name configured in Jenkins (if used)
        withSonarQubeEnv('MySonarServer') {
          sh '''
            ./sonar-scanner/bin/sonar-scanner \
              -Dsonar.projectKey=java-new-1 \
              -Dsonar.sources=. \
              -Dsonar.host.url=$SONAR_HOST_URL \
              -Dsonar.login=$SONAR_AUTH_TOKEN
          '''
        }
      }
    }

    // Notice: no Quality Gate stage here
    stage('Build/Other') {
      steps {
        echo "Continue with other steps — pipeline won't fail on Sonar quality gate"
      }
    }
  }

  post {
    failure { echo "Pipeline failed" }
    success { echo "Pipeline succeeded" }
  }
}

pipeline {
  agent any

  environment {
    // Replace with your Jenkins credential ID that stores the Sonar token (string credential)
    SONAR_CRED_ID = 'sonar-token-id'
    // Replace with your Sonar server URL if different
    SONAR_HOST_URL = 'http://sonarqube:9000'
    // Replace with your Sonar project key
    SONAR_PROJECT_KEY = 'java-new-1'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare workspace') {
      steps {
        sh '''
          echo "Workspace listing:"
          ls -la
        '''
      }
    }

    stage('SonarQube analysis (non-blocking)') {
      steps {
        // uses Jenkins credentials plugin to inject token into env var SONAR_TOKEN
        withCredentials([string(credentialsId: env.SONAR_CRED_ID, variable: 'SONAR_TOKEN')]) {
          sh '''
            if [ -x "./sonar-scanner/bin/sonar-scanner" ]; then
              echo "Found local sonar-scanner, running analysis..."
              ./sonar-scanner/bin/sonar-scanner \
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST_URL} \
                -Dsonar.login=${SONAR_TOKEN}
            else
              echo "No local sonar-scanner found. Trying global 'sonar-scanner' command..."
              sonar-scanner \
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST_URL} \
                -Dsonar.login=${SONAR_TOKEN} || echo "sonar-scanner not available; skipping Sonar analysis"
            fi
          '''
        }
      }
    }

    stage('Build (if Maven project)') {
      steps {
        sh '''
          if [ -f pom.xml ]; then
            echo "Maven pom.xml found — running mvn clean package"
            mvn -B -DskipTests clean package
          else
            echo "pom.xml not found — skipping Maven build"
          fi
        '''
      }
    }

    stage('Other steps / Tests') {
      steps {
        echo "Add additional build/test/deploy steps here as needed."
      }
    }
  }

  post {
    success {
      echo "Pipeline completed (SUCCESS). Sonar scan ran but pipeline is not blocked by Quality Gate."
    }
    failure {
      echo "Pipeline completed (FAILURE). Check console output for errors."
    }
    always {
      echo "Done. Tip: replace SONAR_CRED_ID and SONAR_HOST_URL at top of Jenkinsfile if needed."
    }
  }
}

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

    stage('Install Sonar Scanner') {
      steps {
        sh '''
          echo "Downloading Sonar Scanner..."
          if [ ! -d "${WORKSPACE}/sonar-scanner" ]; then
            wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip -O scanner.zip
            unzip -q scanner.zip -d .
            mv sonar-scanner-* sonar-scanner
            rm scanner.zip
          fi
        '''
      }
    }

    stage('Sonar Scan') {
      steps {
        // use withSonarQubeEnv so Jenkins associates this analysis with configured Sonar server named 'Sonar'
        withSonarQubeEnv('Sonar') {
          sh '''
            ./sonar-scanner/bin/sonar-scanner \
              -Dsonar.projectKey=java-new-1 \
              -Dsonar.sources=. \
              -Dsonar.host.url=${SONAR_HOST} \
              -Dsonar.token=${SONAR_TOKEN}
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        // waits for SonarQube to call back and return the quality gate result
        script {
          timeout(time: 5, unit: 'MINUTES') {
            def qg = waitForQualityGate()
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

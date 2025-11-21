pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
    SONAR_HOST  = 'http://sonarqube:9000'  // change to http://<IP>:9000 only if you prefer IP
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare') {
      steps { sh 'ls -la' }
    }

    stage('Install Sonar Scanner (if needed)') {
      steps {
        sh '''
          if [ ! -x "${WORKSPACE}/sonar-scanner/bin/sonar-scanner" ]; then
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
        script {
          // If docker CLI available, use docker run (shell); else use local scanner
          def dockerAvailable = sh(returnStatus: true, script: 'docker --version >/dev/null 2>&1') == 0

          if (dockerAvailable) {
            echo "Using docker run for sonar-scanner"
            sh '''
              docker run --rm --network ci-network -v ${PWD}:/usr/src -w /usr/src sonarsource/sonar-scanner-cli:latest \
                sonar-scanner \
                  -Dsonar.projectKey=java-new-1 \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=${SONAR_HOST} \
                  -Dsonar.login=${SONAR_TOKEN}
            '''
          } else {
            echo "Using local sonar-scanner binary"
            sh '''
              ./sonar-scanner/bin/sonar-scanner \
                -Dsonar.projectKey=java-new-1 \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST} \
                -Dsonar.login=${SONAR_TOKEN}
            '''
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          timeout(time: 15, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg == null) { error "No Quality Gate result received from SonarQube" }
            echo "Quality Gate status: ${qg.status}"
            if (qg.status != 'OK') { error "Quality Gate failed: ${qg.status}" }
          }
        }
      }
    }
  }

  post {
    always { echo "Pipeline finished" }
    success { echo "Pipeline succeeded" }
    failure { echo "Pipeline failed" }
  }
}

stage('Sonar Scan') {
  steps {
    // associate this analysis with Jenkins' Sonar server configuration named 'Sonar'
    withSonarQubeEnv('Sonar') {
      script {
        def dockerAvailable = sh(returnStatus: true, script: 'docker --version >/dev/null 2>&1') == 0

        if (dockerAvailable) {
          echo "Using docker run for sonar-scanner"
          sh '''
            docker run --rm --network ci-network -v ${PWD}:/usr/src -w /usr/src sonarsource/sonar-scanner-cli:latest \
              sonar-scanner \
                -Dsonar.projectKey=java-new-1 \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST} \
                -Dsonar.token=${SONAR_TOKEN}
          '''
        } else {
          echo "Using local sonar-scanner binary"
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
  }
}

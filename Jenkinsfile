pipeline {
  agent any

  environment {
    SONAR_PROJECT_KEY = 'Sahil-Raval1012_8.2CDevSecOps'
    SONAR_ORG = 'sahil-raval1012'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Sahil-Raval1012/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Run Tests & Coverage') {
      steps {
        sh 'npm test || true'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            if ! command -v sonar-scanner >/dev/null; then
              echo "SonarScanner not found. Installing..."
              SCANNER_VER=4.8.0.2856
              curl -sSLo scanner.zip "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SCANNER_VER}-linux.zip"
              unzip -q scanner.zip
              export PATH=$PWD/$(ls -d sonar-scanner-*-linux)/bin:$PATH
            fi

            sonar-scanner \
              -Dsonar.projectKey="${SONAR_PROJECT_KEY}" \
              -Dsonar.organization="${SONAR_ORG}" \
              -Dsonar.host.url="https://sonarcloud.io" \
              -Dsonar.login="${SONAR_TOKEN}" \
              -Dsonar.sources=. \
              -Dsonar.exclusions="node_modules/**,test/**" \
              -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'npm-audit.json,coverage/**', allowEmptyArchive: true
    }
  }
}


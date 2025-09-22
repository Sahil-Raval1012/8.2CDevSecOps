pipeline {
  agent any

  environment {
    GIT_REPO = 'https://github.com/your_github_username/8.2CDevSecOps.git'
    SONAR_PROJECT_KEY = 'yourname_8.2CDevSecOps'   // replace
    SONAR_ORG = 'your_sonarcloud_org'             // replace
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: env.GIT_REPO
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test || true'   // allow tests to fail but continue pipeline
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // ensure your package.json has a coverage script (e.g., nyc mocha)
        sh 'npm run coverage || true'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh 'npm audit --json > npm-audit.json || true'
        sh 'cat npm-audit.json || true'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        // SONAR_TOKEN must be added into Jenkins credentials with ID 'SONAR_TOKEN'
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            set -e
            # --- download sonar-scanner CLI (adjust version if needed) ---
            SCANNER_VER=4.8.0.2856
            curl -sSLo sonar-scanner.zip "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SCANNER_VER}-linux.zip"
            unzip -q sonar-scanner.zip
            SCANNER_DIR=$(ls -d sonar-scanner-*-linux || true)
            export PATH=$PWD/${SCANNER_DIR}/bin:$PATH

            # confirm coverage file path used by sonar
            if [ -f coverage/lcov.info ]; then
              echo "lcov found"
            else
              echo "lcov missing - continuing anyway"
            fi

            # run sonarscanner
            sonar-scanner \
              -Dsonar.projectKey="${SONAR_PROJECT_KEY}" \
              -Dsonar.organization="${SONAR_ORG}" \
              -Dsonar.host.url="https://sonarcloud.io" \
              -Dsonar.login="${SONAR_TOKEN}" \
              -Dsonar.sources="." \
              -Dsonar.exclusions="node_modules/**,test/**" \
              -Dsonar.javascript.lcov.reportPaths="coverage/lcov.info"
          '''
        }
      }
    }
  }

  post {
    always {
      // save files you need for the report/video
      archiveArtifacts artifacts: 'npm-audit.json,coverage/**,sonar-scanner.zip', allowEmptyArchive: true
    }
  }
}


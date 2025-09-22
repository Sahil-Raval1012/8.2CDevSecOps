pipeline {
    agent any
    
    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:$PATH"
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        SCANNER_VERSION = '4.7.0.2747'
        SCANNER_HOME = "${WORKSPACE}/sonar-scanner-${SCANNER_VERSION}-macosx"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sahil-Raval1012/8.2CDevSecOps.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "=== Environment Check ==="
                    echo "Node.js location: $(which node)"
                    echo "npm location: $(which npm)"
                    echo "Node version: $(node --version)"
                    echo "npm version: $(npm --version)"
                    echo "Current directory: $(pwd)"
                    
                    echo "=== Installing Dependencies ==="
                    npm install
                    echo "Dependencies installed successfully"
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    echo "=== Running Tests ==="
                    npm test || true
                '''
            }
        }
        
        stage('Generate Coverage Report') {
            steps {
                sh '''
                    echo "=== Generating Coverage Report ==="
                    npm run coverage || true
                    
                    # Check if coverage report was generated
                    if [ -f "coverage/lcov.info" ]; then
                        echo "Coverage report generated successfully"
                        echo "Coverage file size: $(du -h coverage/lcov.info)"
                    else
                        echo "Warning: Coverage report not found"
                    fi
                '''
            }
        }
        
        stage('NPM Audit (Security Scan)') {
            steps {
                sh '''
                    echo "=== Running NPM Security Audit ==="
                    npm audit --audit-level=high || true
                    
                    echo "=== NPM Audit Summary ==="
                    npm audit --parseable | head -20 || true
                '''
            }
        }
        
        stage('SonarCloud Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        echo "=== Starting SonarCloud Analysis ==="
                        
                        # Clean up any existing scanner installations
                        rm -rf sonar-scanner-*
                        
                        echo "=== Downloading SonarScanner CLI ==="
                        curl -o sonar-scanner-cli-${SCANNER_VERSION}-macosx.zip \
                             -L https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SCANNER_VERSION}-macosx.zip
                        
                        echo "=== Extracting SonarScanner ==="
                        unzip -o sonar-scanner-cli-${SCANNER_VERSION}-macosx.zip
                        
                        # Make scanner executable
                        chmod +x ${SCANNER_HOME}/bin/sonar-scanner
                        
                        echo "=== Verifying SonarScanner Installation ==="
                        ${SCANNER_HOME}/bin/sonar-scanner --version
                        
                        echo "=== Running SonarCloud Analysis ==="
                        ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=sahil-raval1012_8.2CDevSecOps \
                            -Dsonar.organization=sahil-raval1012 \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=node_modules/**,test/**,sonar-scanner-*,coverage/** \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                            -Dsonar.projectName="NodeJS Goof Vulnerable App" \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.verbose=true
                        
                        echo "=== SonarCloud Analysis Completed ==="
                        echo "Check your SonarCloud dashboard at: https://sonarcloud.io/project/overview?id=sahil-raval1012_8.2CDevSecOps"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "=== Pipeline Execution Summary ==="
            echo "Pipeline completed at: ${new Date()}"
            
            // Clean up downloaded scanner files to save space
            sh 'rm -rf sonar-scanner-* || true'
        }
        success {
            echo "=== Pipeline Succeeded ==="
            echo "All stages completed successfully!"
        }
        failure {
            echo "=== Pipeline Failed ==="
            echo "Check the console output for error details"
        }
    }
}

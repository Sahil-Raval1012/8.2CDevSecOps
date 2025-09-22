pipeline {
    agent any
    
    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:$PATH"
        SONAR_TOKEN = credentials('SONAR_TOKEN')
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
                    echo "Node.js location:"
                    which node
                    echo "npm location:"
                    which npm
                    echo "Node version:"
                    node --version
                    echo "npm version:"
                    npm --version
                    echo "Installing dependencies..."
                    npm install
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
        }
        
        stage('Generate Coverage Report') {
            steps {
                sh 'npm run coverage || true'
            }
        }
        
        stage('NPM Audit (Security Scan)') {
            steps {
                sh 'npm audit || true'
            }
        }
        
        stage('SonarCloud Analysis') {
            steps {
                sh '''
                    echo "Starting SonarCloud analysis..."
                    
                    # Remove any existing scanner to avoid conflicts
                    rm -rf sonar-scanner-*
                    
                    # Use a compatible SonarScanner version for Java 11
                    curl -o sonar-scanner-cli-4.7.0.2747-macosx.zip -L https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-macosx.zip
                    
                    # Extract with overwrite flag to avoid prompts
                    unzip -o sonar-scanner-cli-4.7.0.2747-macosx.zip
                    
                    # Make scanner executable
                    chmod +x sonar-scanner-4.7.0.2747-macosx/bin/sonar-scanner
                    
                    # Run SonarCloud analysis
                    ./sonar-scanner-4.7.0.2747-macosx/bin/sonar-scanner \
                        -Dsonar.projectKey=sahil-raval1012_8.2CDevSecOps \
                        -Dsonar.organization=sahil-raval1012 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,test/**,sonar-scanner-* \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                        -Dsonar.projectName="NodeJS Goof Vulnerable App" \
                        -Dsonar.sourceEncoding=UTF-8
                    
                    echo "SonarCloud analysis completed"
                '''
            }
        }
    }
}

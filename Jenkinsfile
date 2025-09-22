pipeline {
    agent any
    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:$PATH"
    }
    stages {
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
    }
}

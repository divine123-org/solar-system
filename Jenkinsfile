pipeline {
    agent any

    tools {
        // the name of the nodejs configuration in Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-17-0-1'
    }

    stages {
        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('NPM Dependency Audit') {
            steps {
                // check crutial dependency vunurabilities from package.json using NPM audit. Fail build if exits
                sh 'npm audit --audit-level=critical'
            }
        }
    }
}
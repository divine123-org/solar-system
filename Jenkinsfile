pipeline {
    agent any

    tools {
        // the name of the nodejs configuration in Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-23-8-0'
    }

    stages {
        stage('VM Node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
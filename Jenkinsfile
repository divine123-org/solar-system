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

        stage('Dependency Scanning') {
            // Scan the dependency using 2 methods in parallel
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        // check crutial dependency vunurabilities from package.json using NPM audit. Fail build if exits
                        sh 'npm audit --audit-level=critical'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        // check crutial dependency vunurabilities from package.json using 3rd party tool, OWAS depencency check plugin (for nodejs)
                        // Check doc for more details on command
                        depencencyCheck additionalArguments: '''                
                        --scan \'./\'
                        --out \'./\'
                        --format \'ALL\'
                        --prettyPrint 
                        ''', odcInstallation: 'OWAS-Dependency-10' 
                    }
                }
            }
        }

        
    }
}
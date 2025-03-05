pipeline {
    agent any

    tools {
        // the name of the nodejs configuration in Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-17-0-1'
    }

    // environment {
    //     MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    // }

    stages {
        stage('Installing Dependencies') {
            options { timestamps() }
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Scanning') {
            // Scan the dependency using 2 methods in parallel.
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        // check crutial dependency vulnerabilities from package.json using NPM audit. Fail build if exits
                        sh 'npm audit --audit-level=critical'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        // check crutial dependency vunurabilities from package.json using 3rd party tool, OWAS depencency check plugin (for nodejs)
                        // Check doc for more details on how to use the command
                        dependencyCheck additionalArguments: '''                
                            --scan \'./\'
                            --out \'./\'
                            --format \'ALL\'
                            --prettyPrint 
                        ''', 
                        odcInstallation: 'OWAS-Dependency-10' 
                        // dependencyCheckPublisher failedTotalCritical: 1, pattern: '', stopBuild: true
                        // junit <complete these two>
                        // publishHTML
                    }
                }
            }
        }

        // stage('Unit Testing') {
        //     options { retry(2) }
        //     steps {
        //         // create a credential in jenkins ui and set the credentialId here
        //         withCredentials([usernamePassword(credentialsId: '', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm test'
        //         }
        //         // Publish the output of all the test cases
        //     }
        // }
    }
}
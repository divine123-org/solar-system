pipeline {
    agent any

    tools {
        // the name of the nodejs configuration in Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-23-9-0'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }

    stages {
        stage('Installing Dependencies') {
            options { timestamps() }
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        // Fail the build if critical vunerability exist
                        sh 'npm audit --audit-level=critical'
                    }
                }

                stage('Snyk Security Scan') {
                    steps {
                        snykSecurity( 
                            snykInstallation: 'Snyk-latest', // Please define a Snyk installation in the Jenkins Global Tool Configuration. This task will not run without a Snyk installation, obviously.
                            snykTokenId: 'snyk-cli-token', 
                            additionalArguments: 'snyk test --severity-threshold=critical -d --fail-fast --file=package.json --file=package-lock.json', // Run in Debug Mode to See What’s Happening. Look for slow steps in the logs.
                            )
                    }
                }
            }
        }

        // stage('Unit Testing') {
        //     options { retry(2) }
        //     steps {
        //         // create a credential in jenkins ui and set the credentialId here
        //         withCredentials([usernamePassword(credentialsId: 'mongodb-credential-id', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm test'
        //         }
        //         // Publish the output of all the test cases
        //     }
        // }

        // stage('Code Coverage') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'mongodb-credential-id', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm run coverage'
        //         }
        //     }
        // }
    }
}
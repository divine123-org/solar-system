pipeline {
    agent any

    tools {
        // the name of the configuration Tools from Jenkins. Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-23-9-0'
        snyk 'Snyk-latest'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }

    stages {
        stage('Installing Dependencies') {
            options { timestamps() }
            steps {
                sh 'npm install --no-audit'
                sh 'npm install snyk-to-html -g'
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
                        withCredentials([string(credentialsId: 'snyk-cli-token', variable: 'SNYK_API_KEY')]) {
                            script {
                                // Ensure Snyk is available
                                sh 'which snyk || npm install -g snyk'

                                // Debug token and auth
                                sh 'echo "SNYK_API_KEY is set" || echo "SNYK_API_KEY is empty"'
                                sh 'snyk auth $SNYK_API_KEY || true'  // Authenticate with token, ignore failure

                                // Run Snyk test
                                sh 'snyk test --severity-threshold=critical --fail-on=upgradable -d --json | tee snyk_report.json'
                                
                                // Always generate HTML report
                                sh 'snyk-to-html -i snyk_report.json -o snyk_dependency_check_report.html'
                            }
                        }
                    }
                    post {
                        always {
                            // Archive Snyk JSON Report and HTML Report
                            archiveArtifacts artifacts: 'snyk_report.json', allowEmptyArchive: true,onlyIfSuccessful: false
                            archiveArtifacts artifacts: 'snyk_dependency_check_report.html', allowEmptyArchive: true, onlyIfSuccessful: false                            
                        }
                        success {
                            echo '✅ Build passed no security vunerability!'
                        }
                        failure {
                            echo '❌ Build failed due to security vulnerabilities!'
                        }
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
        //         // create a credential in jenkins ui and set the credentialId here
        //         withCredentials([usernamePassword(credentialsId: 'mongodb-credential-id', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {            
                sh 'npm test'
        //         }
        //         // Publish the output of all the test cases
            }
        }

        // stage('Code Coverage') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'mongodb-credential-id', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm run coverage'
        //         }
        //     }
        // }
    }
}
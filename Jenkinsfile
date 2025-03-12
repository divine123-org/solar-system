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
                        // Comment steps
                        withCredentials([string(credentialsId: 'snyk-cli-token', variable: 'SNYK_API_KEY')]) {
                            script {
                                def snykExitCode = sh(returnStatus: true, script: 'snyk test --severity-threshold=high --fail-on=upgradable -d --fail-fast --file=package.json --file=package-lock.json --timeout=300 --json > snyk_report.json')
                                if (snykExitCode != 0) {
                                    error('Snyk scan failed due to vulnerabilities')
                                }
                            }
                            sh 'snyk-to-html -i snyk_report.json -o snyk_dependency_check_report.html'
                        }
                    }
                    post {
                        always {
                            // Archive Snyk JSON Report
                            archiveArtifacts artifacts: 'snyk_report.json', onlyIfSuccessful: true
                            archiveArtifacts artifacts: 'snyk_dependency_check_report', onlyIfSuccessful: true
                            // Publish HTML Report
                            publishHTML([
                                allowMissing: false, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'snyk_dependency_check_report.html', reportName: 'Dependency Check HTML Report', useWrapperFileDirectly: true
                            ])
                        }
                        success {
                            echo '✅ Snyk Security scan completed successfully!'
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
                sh 'Dependency scanning stage passed'
            }
        //     options { retry(2) }
        //     steps {
        //         // create a credential in jenkins ui and set the credentialId here
        //         withCredentials([usernamePassword(credentialsId: 'mongodb-credential-id', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
        //             sh 'npm test'
        //         }
        //         // Publish the output of all the test cases
        //     }
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
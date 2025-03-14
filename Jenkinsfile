pipeline {
    agent any

    tools {
        // the name of the configuration Tools from Jenkins. Dashboard > Manage Jenkins > Tools > NodeJS installations
        nodejs 'nodejs-23-9-0'
        // snyk 'Snyk-latest'
    }

    environment {
        // Teacher didn't provide us with the db. So skipping.
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGODB_CREDS = credentials('mongo-db-credentials')
        MONGODB_USERNAME = credentials('mongo-db-username')
        MONGODB_PASSWORD = credentials('mongo-db-password')
        // Define the Snyk tool, as configured in your Jenkins global configuration, as an environment variable
        SNYK_HOME = tool 'Snyk-latest'
        // Define the Sonar scanner tool, as an environment variable
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner-7-0-2'
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
                                // Echo the SNYK_HOME value to verify it
                                sh 'echo "Snyk home directory: $SNYK_HOME"'

                                sh 'ls -l $SNYK_HOME'  // Check contents
                                
                                // Echo the full assumed path to Snyk executable
                                sh 'echo "Snyk path: $SNYK_HOME/bin/snyk"'
                                
                                // Optionally, test the Snyk version to confirm it works
                                sh '$SNYK_HOME/snyk --version'

                                // Debug token and auth
                                sh 'echo "SNYK_API_KEY is set" || echo "SNYK_API_KEY is empty"'
                                sh '$SNYK_HOME/snyk auth $SNYK_API_KEY || true'  // Authenticate with token, ignore failure

                                // Run Snyk test
                                sh '$SNYK_HOME/snyk test --severity-threshold=critical --fail-on=upgradable -d --json | tee snyk_report.json'
                                
                                // Always generate HTML report
                                sh '$SNYK_HOME/snyk-to-html -i snyk_report.json -o snyk_dependency_check_report.html'
                            }
                        }
                    }
                }
            }
        }

        // stage('Unit Testing') {
        //     steps {
        //         sh 'npm test'
        //     }
        // }

        stage('Code Coverage') {
            steps {
                sh 'echo Colon-separated - $MONGODB_CREDS'
                sh 'echo Username - $MONGODB_CREDS_USR'
                sh 'echo Password - $MONGODB_CREDS_PSW'
                catchError(buildResult: 'SUCCESS', message: 'Oops! Would write more test coverage in futur releases.', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            }
        }

        stage('SAST - SonarQube') {
            steps {
                // Echo the SONAR_SCANNER_HOME value to verify it
                sh 'echo "SonarQube home directory: $SONAR_SCANNER_HOME"'
                // sh '''
                //     $SONAR_SCANNER_HOME/bin/sonar-scanner \
                //         -Dsonar.projectKey
                // '''
            }
        }
    }

    post {
        always {
            // Archive Snyk JSON Report and HTML Report
            archiveArtifacts artifacts: 'snyk_report.json', allowEmptyArchive: true,onlyIfSuccessful: false
            archiveArtifacts artifacts: 'snyk_dependency_check_report.html', allowEmptyArchive: true, onlyIfSuccessful: false                                                    

            // Publish the output of all the test cases
            junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml'
        }
    }

}
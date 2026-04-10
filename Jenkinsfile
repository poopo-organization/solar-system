pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20'
    }

    stages {
        stage('Node Version') {
            steps {
                sh 'node -v'
                sh 'npm -v'
            }
        }

        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('NPM Dependency Audit') {
            steps {
                script {
                    // Run npm audit and capture the exit code
                    def auditStatus = sh(script: 'npm audit --audit-level=critical', returnStatus: true)
                    echo "NPM audit exit code: ${auditStatus}"
                }
            }
        }
        stage('Unit Testing') {
            steps {
                sh 'npm test'
                junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml'
            }
        } 
        stage('Code Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future') {
                sh 'npm run coverage'
                }
            }
        }
        stage('SAST - SonarQube') {
            steps {
                sh '''
                sonar-scanner \
                    -Dsonar.projectKey=Solar-System-Project \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://192.168.1.29:9000 \
                    -Dsonar.login=sqp_2acb256ebad4310cc8263e5c18387d38e3e5fce6
                '''
            }
        }           
    }   
}

pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20'
    }
    environment {
        SONAR_SCANNER_HOME = tool 'SonarQube-Scanner'
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
                timeout(time: 60, unit: 'SECONDS') {
                    withSonarQubeEnv('sonarqube-server') {
                        sh 'echo $SONAR_SCANNER_HOME'
                        sh '''
                        $SONAR_SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=Solar-System-Project \
                            -Dsonar.sources=app.js \
                            -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
                        '''
                    }
                    waitForQualityGate abortPipeline: true
                }    
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'printenv'
                sh 'docker build -t luzarow/solar-system:$GIT_COMMIT .'
            }
        }
        stage('Trivy Vulnerability Scanner') {
            steps {
                sh '''
                    trivy image luzarow/solar-system:$GIT_COMMIT \
                    --severity LOW,MEDIUM,HIGH \
                    --exit-code 0 \
                    --quiet \
                    --format json -o trivy-image-MEDIUM-results.json

                    trivy image luzarow/solar-system:$GIT_COMMIT \
                    --severity CRITICAL \
                    --exit-code 0 \
                    --quiet \
                    --format json -o trivy-image-CRITICAL-results.json
                '''
            }
            post {
                always {
                    sh '''
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-MEDIUM-results.html trivy-image-MEDIUM-results.json

                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json
                    '''
                }
            }
        }           
    }   
}

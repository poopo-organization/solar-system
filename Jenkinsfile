pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20'
    }
    environment {
        GITHUB_TOKEN = credentials('github-api-token')
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
                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-cred', url: "") {
                    sh 'docker push luzarow/solar-system:$GIT_COMMIT'
                }
            }
        }
        stage('K8S - Update Image Tag') {
            steps {
                // Clone the new repository
                sh 'git clone -b main https://github.com/poopo-organization/solar-system-argoCD.git'
                
                dir("solar-system-argoCD/kubernetes") {
                    sh '''
                        #### Replace Docker Tag ####
                        git checkout main
                        git checkout -b feature-$BUILD_ID
                        sed -i "s#luzarow.*#luzarow/solar-system:$GIT_COMMIT#g" deployment.yml
                        cat deployment.yml
                        
                        #### Commit and Push to Feature Branch ####
                        git config --global user.email "jenkins@popo.com"
                        git remote set-url origin https://$GITHUB_TOKEN@github.com/poopo-organization/solar-system-argoCD.git
                        git add .
                        git commit -am "Updated docker image"
                        git push -u origin feature-$BUILD_ID
                    '''
                }
            }
        }           
    }   
}

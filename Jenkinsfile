pipeline {
    agent any

    tools {
        nodejs 'Nodejs-25'
    }

    stages {
        stage('VM Node Version') {
            steps {
                sh 'node -v'
                sh 'npm -v'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        } 

        stage('NPM Dependency Audit') {
            steps {
                sh '''
                    npm audit --audit-level=critical
                    echo $?
                '''
            }       
        }
    }
}
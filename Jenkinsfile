pipeline {
    agent any

    tools {
        nodejs 'Nodejs-25'
    }

    stages {
        stage('VM Node Version') {
            steps {
                // Execute shell commands
                sh 'node -v'
                sh 'npm -v'
            }
        }
    }
}
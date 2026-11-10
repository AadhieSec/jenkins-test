pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checked out!'
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'ls -la'
                sh 'ls -la src/'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing application...'
                sh 'test -f src/index.html'
            }
        }

        stage('Success') {
            steps {
                echo 'Jenkins pipeline completed!'
            }
        }
    }
}

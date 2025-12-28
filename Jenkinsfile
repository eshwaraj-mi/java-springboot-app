pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

    }
}
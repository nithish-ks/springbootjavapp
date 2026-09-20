pipeline {
    agent any

    tools {
        maven 'maven'
    }

    stages {

        stage('Checkout from Git') {
            steps {
                git branch: 'prod',
                    url: 'https://github.com/nithish-ks/springbootjavapp.git'
            }
        }

        stage('Validate with Maven') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Compile with Maven') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test with Maven') {
            steps {
                sh 'mvn test'
            }

            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml',
                          allowEmptyResults: true
                }
            }
        }

        stage('Package with Maven') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
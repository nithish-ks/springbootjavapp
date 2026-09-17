pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                ssh 'mvn clean package'
            }
        }
    }
}
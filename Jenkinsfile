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

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar-server') {
            sh '''
                mvn sonar:sonar \
                    -Dsonar.projectKey=springbootjavaapp \
                    -Dsonar.projectName=springbootjavaapp \
                    -Dsonar.java.binaries=target/classes
            '''
        }
    }
}

        stage('Package with Maven') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Docker Build') {
    steps {
        sh 'docker build -t petclinic:latest .'
    }
}

stage('Trivy Image Scan') {
    steps {
        sh '''
            trivy image --severity HIGH,CRITICAL --format table \
                -o trivy-image-report.txt \
                petclinic:latest
        '''

        archiveArtifacts artifacts: 'trivy-image-report.txt',
                         allowEmptyArchive: true
    }
}
    }
}
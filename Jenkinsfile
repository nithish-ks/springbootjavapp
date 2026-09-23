pipeline {
    agent any

    environment {
    ACR_SERVER = 'acrnithish090826.azurecr.io'
    IMAGE_NAME = 'springbootjavaapp'
    IMAGE_TAG  = 'latest'
    DEPLOYMENT_NAME = 'petclinic'
K8S_NAMESPACE   = 'default'
EMAIL_FROM       = 'chickoo.nithish@gmail.com'
EMAIL_RECIPIENTS = 'nithishks007@gmail.com'
}

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
        sh 'docker build -t $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG .'
    }
}
stage('Trivy Image Scan') {
    steps {
        sh '''
            trivy image --severity HIGH,CRITICAL --format table \
                -o trivy-image-report.txt \
                $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG
        '''

        archiveArtifacts artifacts: 'trivy-image-report.txt',
                         allowEmptyArchive: true
    }
}


stage('Push to ACR') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'acr-creds',
                usernameVariable: 'ACR_USER',
                passwordVariable: 'ACR_PASS'
            )
        ]) {
            sh '''
                echo "$ACR_PASS" | docker login $ACR_SERVER \
                    -u "$ACR_USER" \
                    --password-stdin

                docker push $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG

                docker logout $ACR_SERVER
            '''
        }
    }
}

stage('Deploy to AKS') {
    steps {
        withCredentials([
            file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
        ]) {
            sh '''
                kubectl apply -n $K8S_NAMESPACE -f k8s/deployment.yaml
                kubectl apply -n $K8S_NAMESPACE -f k8s/service.yaml
            '''
        }
    }
}

stage('Verify Deployment Rollout') {
    steps {
        withCredentials([
            file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
        ]) {
            sh '''
                kubectl get pods -n $K8S_NAMESPACE
                kubectl get svc -n $K8S_NAMESPACE
                kubectl rollout status deployment/$DEPLOYMENT_NAME \
                    -n $K8S_NAMESPACE \
                    --timeout=180s
            '''
        }
    }
}

    }

    post {
    success {
        script {
            echo "Deployment verified successfully. Sending success email."

            withCredentials([
                string(
                    credentialsId: 'brevo-api-key',
                    variable: 'BREVO_API_KEY'
                )
            ]) {
                sh """
    HTTP_CODE=\$(curl -s -o /tmp/brevo.out -w '%{http_code}' \
      -X POST https://api.brevo.com/v3/smtp/email \
      -H "api-key: \$BREVO_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "sender": {"email": "${EMAIL_FROM}"},
        "to": [{"email": "${EMAIL_RECIPIENTS}"}],
        "subject": "SUCCESS: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        "textContent": "Pipeline completed successfully and PetClinic was deployed to AKS."
      }')

    echo "Brevo HTTP response: \$HTTP_CODE"
    cat /tmp/brevo.out
"""
            }
        }
    }

    failure {
        script {
            echo "Pipeline failed. Sending failure email."

            withCredentials([
                string(
                    credentialsId: 'brevo-api-key',
                    variable: 'BREVO_API_KEY'
                )
            ]) {
                sh """
                    curl -X POST https://api.brevo.com/v3/smtp/email \
                      -H "api-key: \$BREVO_API_KEY" \
                      -H "Content-Type: application/json" \
                      -d '{
                        "sender": {"email": "${EMAIL_FROM}"},
                        "to": [{"email": "${EMAIL_RECIPIENTS}"}],
                        "subject": "FAILED: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        "textContent": "Pipeline failed. Check Jenkins console log: ${env.BUILD_URL}console"
                      }'
                """
            }
        }
    }

    always {
        echo "Build result: ${currentBuild.currentResult}"
    }
}
}
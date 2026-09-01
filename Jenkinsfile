pipeline {
    agent any

    environment {
        IMAGE_NAME = "eks-test-app"
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'trivy image --severity CRITICAL --exit-code 1 ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
    }
}
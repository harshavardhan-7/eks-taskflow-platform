pipeline {
    agent any

    environment {
        IMAGE_NAME = "eks-test-app"
        PATH = "/opt/homebrew/bin/:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
        AWS_REGION = "us-east-1"
        ECR_REPOSITORY = "eks-test-app"
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
                sh '''
                    /opt/homebrew/bin/trivy image \
                    --scanners vuln \
                    --severity CRITICAL \
                    --ignore-unfixed \
                    --exit-code 1 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        ACCOUNT_ID=$(aws sts get-caller-identity \
                            --query Account \
                            --output text)

                        ECR_REGISTRY="${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                        GIT_SHA=$(git rev-parse --short HEAD)

                        aws ecr get-login-password \
                            --region ${AWS_REGION} \
                        | docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}

                        # Jenkins build-number tag
                        docker tag \
                            ${IMAGE_NAME}:${BUILD_NUMBER} \
                            ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                        # Git commit tag
                        docker tag \
                            ${IMAGE_NAME}:${BUILD_NUMBER} \
                            ${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_SHA}

                        docker push \
                            ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                        docker push \
                            ${ECR_REGISTRY}/${ECR_REPOSITORY}:${GIT_SHA}
                    '''
                }
            }
        }
    }
}
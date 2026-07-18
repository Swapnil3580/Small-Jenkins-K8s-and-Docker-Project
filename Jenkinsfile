pipeline {
    agent any

    environment {
        ACCOUNT_ID = "108632297954"
        AWS_REGION = "ap-southeast-1"
        IMAGE_NAME = "my-devops-project"
        ECR_REPO = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Create Python Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin \
                ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push ${ECR_REPO}:${IMAGE_TAG}
                docker push ${ECR_REPO}:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/flask-app \
                flask-app=${ECR_REPO}:${IMAGE_TAG}

                kubectl rollout status deployment/flask-app --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get deployment
                kubectl get svc
                '''
            }
        }
    }

    post {

        success {
            echo '====================================='
            echo ' CI/CD Pipeline Completed Successfully'
            echo " Docker Image: ${ECR_REPO}:${IMAGE_TAG}"
            echo ' Application deployed to Kubernetes'
            echo '====================================='
        }

        failure {
            echo '====================================='
            echo ' Pipeline Failed'
            echo 'Check Jenkins Console Output'
            echo '====================================='
        }
    }
}

pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '690084475612'
        ECR_REPOSITORY = 'registryimage-myapp'
        ECR_REGISTRY = "690084475612.dkr.ecr.ap-south-1.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test Container') {
            steps {
                sh '''
                    docker run -d \
                    --name test-container \
                    -p 5001:5000 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:5001/health

                    docker stop test-container
                    docker rm test-container
                '''
            }
        }

        stage('AWS Debug') {
            steps {
                 sh '''
                     echo "AWS Region:"
                     echo ${AWS_REGION}

                     echo "AWS Account:"
                     aws sts get-caller-identity

                     echo "ECR repositories:"
                     aws ecr describe-repositories \
                     --region ${AWS_REGION}

                     echo "Target repository:"
                     echo ${ECR_REPO}
                 '''
             }
        }
         stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ap-south-1 | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }
    }
}

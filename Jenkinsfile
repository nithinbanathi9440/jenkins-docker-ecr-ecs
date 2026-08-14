pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '235899055622'

        ECR_REPOSITORY = 'jenkins-docker-app'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_URI = "${ECR_REGISTRY}/${ECR_REPOSITORY}:latest"

        ECS_CLUSTER = 'jenkins-ecr-ecs-cluster'
        ECS_SERVICE = 'jenkins-docker-service'
    }

    stages {

        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${ECR_REPOSITORY}:latest .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag \
                    ${ECR_REPOSITORY}:latest \
                    ${IMAGE_URI}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                    aws ecs update-service \
                    --cluster ${ECS_CLUSTER} \
                    --service ${ECS_SERVICE} \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                '''
            }
        }

        stage('Check ECS Deployment') {
            steps {
                sh '''
                    aws ecs describe-services \
                    --cluster ${ECS_CLUSTER} \
                    --services ${ECS_SERVICE} \
                    --region ${AWS_REGION} \
                    --query "services[0].[status,runningCount,desiredCount]" \
                    --output table
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD deployment completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
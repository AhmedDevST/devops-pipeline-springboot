pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    triggers {
        // Poll GitHub every 5 minutes
        pollSCM('H/5 * * * *')
    }
      environment {
        
        // AWS Configuration
        AWS_REGION = 'eu-north-1'  
        AWS_ACCOUNT_ID = '646895371279'  
        ECR_REPOSITORY = 'demo1'
        ECS_CLUSTER = 'demo-cluster'
        ECS_SERVICE = 'demo1-task-service-f4010tlr'
        ECS_TASK_DEFINITION = 'demo1-task'
        
        // ECR URL
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}" 
    }
    stages {
        stage('checkout') {
            steps {
                git credentialsId: 'jen_dock_git', url: 'https://github.com/AhmedDevST/devops-pipeline-springboot'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

         stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} -f Dockerfile .
                        docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_REPOSITORY}:latest
                    """
                }
            }
        }
         stage('Push to AWS ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                        sh """
                            # Login to ECR
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REGISTRY}
                            
                            # Tag images for ECR
                            docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                            docker tag ${ECR_REPOSITORY}:latest ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                            
                            # Push to ECR
                            docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                            docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                            
                            echo "Image pushed to ECR: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
                        """
                    }
                }
            }
        }
        stage('Deploy to AWS ECS') {
            steps {
                script {
                    withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                        sh """
                            # Update ECS service to use new image
                            aws ecs update-service \
                                --cluster ${ECS_CLUSTER} \
                                --service ${ECS_SERVICE} \
                                --force-new-deployment \
                                --region ${AWS_REGION}
                            
                            echo " ECS service updated. New deployment initiated."
                            
                            aws ecs wait services-stable \
                                --cluster ${ECS_CLUSTER} \
                                --services ${ECS_SERVICE} \
                                --region ${AWS_REGION}
                            
                            echo "Deployment completed successfully!"
                        """
                    }
                }
            }
        }
         stage('Cleanup Local Images') {
            steps {
                script {
                    sh """
                        docker rmi ${ECR_REPOSITORY}:${IMAGE_TAG} || true
                        docker rmi ${ECR_REPOSITORY}:latest || true
                        docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} || true
                        docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest || true
                    """
                }
            }
        }
    }
    post {
        success {
            echo ' Pipeline executed successfully!'
            echo "Application deployed to ECS: ${ECS_CLUSTER}/${ECS_SERVICE}"
            echo " Image: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        }
        failure {
            echo ' Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}
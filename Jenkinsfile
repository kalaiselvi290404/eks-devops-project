pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'eks-web-app'
        ECR_REGISTRY = '479403398095.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER = 'eks-devops-cluster'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kalaiselvi290404/eks-devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} |
                docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${EKS_CLUSTER}

                sed -i "s|image:.*|image: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}|" kubernetes/deployment.yaml

                kubectl apply -f kubernetes/deployment.yaml
                kubectl apply -f kubernetes/service.yaml

                kubectl rollout status deployment/eks-web-app
                '''
            }
        }
    }
}

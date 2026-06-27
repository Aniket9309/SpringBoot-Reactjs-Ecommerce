pipeline {
    agent { label 'slave' }

    environment {
        AWS_REGION       = 'ap-south-1'
        ECR_REGISTRY     = '198452821908.dkr.ecr.ap-south-1.amazonaws.com'
        BACKEND_REPO     = 'ecommerce-backend'
        FRONTEND_REPO    = 'ecoomerce-frontend'
        IMAGE_TAG        = "${BUILD_NUMBER}"
        NAMESPACE        = 'dev'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Aniket9309/SpringBoot-Reactjs-Ecommerce.git'
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Build & Push Backend') {
            steps {
                sh '''
                    docker build -t $ECR_REGISTRY/$BACKEND_REPO:$IMAGE_TAG ./Ecommerce-Backend
                    docker tag $ECR_REGISTRY/$BACKEND_REPO:$IMAGE_TAG $ECR_REGISTRY/$BACKEND_REPO:latest
                    docker push $ECR_REGISTRY/$BACKEND_REPO:$IMAGE_TAG
                    docker push $ECR_REGISTRY/$BACKEND_REPO:latest
                '''
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                    docker build -t $ECR_REGISTRY/$FRONTEND_REPO:$IMAGE_TAG ./Ecommerce-Frontend
                    docker tag $ECR_REGISTRY/$FRONTEND_REPO:$IMAGE_TAG $ECR_REGISTRY/$FRONTEND_REPO:latest
                    docker push $ECR_REGISTRY/$FRONTEND_REPO:$IMAGE_TAG
                    docker push $ECR_REGISTRY/$FRONTEND_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name my-eks-cluster
                    kubectl apply -f k8s/db-secret.yaml
                    kubectl set image deployment/ecommerce-backend ecommerce-backend=$ECR_REGISTRY/$BACKEND_REPO:$IMAGE_TAG -n $NAMESPACE
                    kubectl set image deployment/ecommerce-frontend ecommerce-frontend=$ECR_REGISTRY/$FRONTEND_REPO:$IMAGE_TAG -n $NAMESPACE
                    kubectl rollout status deployment/ecommerce-backend -n $NAMESPACE
                    kubectl rollout status deployment/ecommerce-frontend -n $NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully! Build #${IMAGE_TAG} deployed to ${NAMESPACE}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}

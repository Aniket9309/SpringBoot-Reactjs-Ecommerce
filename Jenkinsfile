pipeline {
    agent { label 'slave' }

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'qa', 'uat', 'prod'],
            description: 'Select environment to deploy'
        )
    }

    environment {
        AWS_REGION       = 'ap-south-1'
        ECR_REGISTRY     = '198452821908.dkr.ecr.ap-south-1.amazonaws.com'
        BACKEND_REPO     = 'ecommerce-backend'
        FRONTEND_REPO    = 'ecoomerce-frontend'
        IMAGE_TAG        = "${BUILD_NUMBER}"
        NAMESPACE        = "${params.DEPLOY_ENV}"
        SNS_TOPIC        = 'arn:aws:sns:ap-south-1:198452821908:ecommerce-alert'
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

        stage('Ensure Namespace & Secret') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name my-eks-cluster
                    kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE
                    sed "s/namespace: dev/namespace: $NAMESPACE/" k8s/db-secret.yaml | kubectl apply -f -
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    kubectl get deployment ecommerce-backend -n $NAMESPACE || sed "s/namespace: dev/namespace: $NAMESPACE/" k8s/backend-deployment.yaml | kubectl apply -f -
                    kubectl get deployment ecommerce-frontend -n $NAMESPACE || sed "s/namespace: dev/namespace: $NAMESPACE/" k8s/frontend-deployment.yaml | kubectl apply -f -

                    kubectl set image deployment/ecommerce-backend ecommerce-backend=$ECR_REGISTRY/$BACKEND_REPO:$IMAGE_TAG -n $NAMESPACE
                    kubectl set image deployment/ecommerce-frontend ecommerce-frontend=$ECR_REGISTRY/$FRONTEND_REPO:$IMAGE_TAG -n $NAMESPACE

                    kubectl rollout status deployment/ecommerce-backend -n $NAMESPACE --timeout=300s
                    kubectl rollout status deployment/ecommerce-frontend -n $NAMESPACE --timeout=300s
                '''
            }
        }
    }

    post {
        success {
            sh """
                aws sns publish \
                  --topic-arn ${SNS_TOPIC} \
                  --message '✅ Build #${BUILD_NUMBER} deployed successfully to ${params.DEPLOY_ENV} environment!' \
                  --subject 'Jenkins Build SUCCESS - Build #${BUILD_NUMBER}' \
                  --region ${AWS_REGION}
            """
            echo "✅ Build #${BUILD_NUMBER} deployed successfully to ${params.DEPLOY_ENV}"
        }
        failure {
            sh """
                aws sns publish \
                  --topic-arn ${SNS_TOPIC} \
                  --message '❌ Build #${BUILD_NUMBER} FAILED for ${params.DEPLOY_ENV} environment! Check Jenkins logs.' \
                  --subject 'Jenkins Build FAILED - Build #${BUILD_NUMBER}' \
                  --region ${AWS_REGION}
            """
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}

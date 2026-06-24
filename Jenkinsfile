pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT_ID = '198452821908'
        BACKEND_REPO   = 'my-app'
        FRONTEND_REPO  = 'ecommerce-frontend'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube scan on backend...'
                dir('Ecommerce-Backend') {
                    withSonarQubeEnv('MySonarQube') {
                        sh '''
                            mvn clean verify sonar:sonar \
                              -DskipTests \
                              -Dsonar.projectKey=ecommerce-backend
                        '''
                    }
                }
            }
        }

        stage('Build Backend (Maven)') {
            steps {
                echo 'Building Spring Boot backend...'
                dir('Ecommerce-Backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build - Backend') {
            steps {
                echo 'Building backend Docker image...'
                dir('Ecommerce-Backend') {
                    sh "docker build -t ${BACKEND_REPO}:latest ."
                }
            }
        }

        stage('Docker Build - Frontend') {
            steps {
                echo 'Building frontend Docker image...'
                dir('Ecommerce-Frontend') {
                    sh "docker build -t ${FRONTEND_REPO}:latest ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                echo 'Logging in to ECR and pushing images...'
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker tag ${BACKEND_REPO}:latest ${ECR_REGISTRY}/${BACKEND_REPO}:latest
                    docker push ${ECR_REGISTRY}/${BACKEND_REPO}:latest

                    docker tag ${FRONTEND_REPO}:latest ${ECR_REGISTRY}/${FRONTEND_REPO}:latest
                    docker push ${ECR_REGISTRY}/${FRONTEND_REPO}:latest
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Both images pushed to ECR.'
        }
        failure {
            echo 'Pipeline failed! Check the logs above.'
        }
    }
}

pipeline {
    agent any
    
    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }
    
    stages {
        stage('Clone Code') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'cd Ecommerce-Backend && mvn clean package -DskipTests'
            }
        }
        
        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh 'cd Ecommerce-Backend && docker build -t ecommerce-backend:latest .'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    docker stop ecommerce-app || true
                    docker rm ecommerce-app || true
                    docker run -d -p 9090:8080 --name ecommerce-app ecommerce-backend:latest
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

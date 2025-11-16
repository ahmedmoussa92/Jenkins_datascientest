pipeline {
    agent any
    environment {
        // Make sure you have a Jenkins Secret Text or Username/Password credential for Docker Hub
        DOCKERHUB_USER = "ahmedmoussa92"
        DOCKERHUB_PASS = credentials('dockerhub-token') 
    }
    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out source code..."
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo "🚀 Building cast-service image..."
                    sh 'docker build -t $DOCKERHUB_USER/cast-service:latest ./cast-service'

                    echo "🚀 Building movie-service image..."
                    sh 'docker build -t $DOCKERHUB_USER/movie-service:latest ./movie-service'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo "🔐 Logging in to Docker Hub..."
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'

                    echo "📤 Pushing cast-service..."
                    sh 'docker push $DOCKERHUB_USER/cast-service:latest'

                    echo "📤 Pushing movie-service..."
                    sh 'docker push $DOCKERHUB_USER/movie-service:latest'
                }
            }
        }
    }
}
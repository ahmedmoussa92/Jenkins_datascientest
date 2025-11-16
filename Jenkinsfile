pipeline {
    agent any
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
                    sh 'docker build -t ahmedmoussa92/cast-service:latest ./cast-service'

                    echo "🚀 Building movie-service image..."
                    sh 'docker build -t ahmedmoussa92/movie-service:latest ./movie-service'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-token', 
                                                      usernameVariable: 'DOCKERHUB_USER', 
                                                      passwordVariable: 'DOCKERHUB_PASS')]) {
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
}

pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "ahmedmoussa92"
        DOCKERHUB_PASS = credentials('dockerhub-token') // Jenkins Secret Text or Username+Password token
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout source code from GitHub (configured in Jenkins job)
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
                    // Log in to Docker Hub using credentials
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    echo "📤 Pushing cast-service..."
                    sh 'docker push $DOCKERHUB_USER/cast-service:latest'
                    echo "📤 Pushing movie-service..."
                    sh 'docker push $DOCKERHUB_USER/movie-service:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Use Jenkins Secret File 'kubeconfig-dev' to connect to Kubernetes
                    withKubeConfig([credentialsId: 'kubeconfig-dev']) {
                        sh '''
                        helm upgrade --install myapp-dev ./helm/app-chart \
                            -n dev \
                            -f ./helm/app-chart/values-dev.yaml
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker images..."
            sh 'docker system prune -f'
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
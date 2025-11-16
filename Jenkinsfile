pipeline {
    agent any
    environment {
        // Ensure you have a Jenkins Secret Text credential named 'dockerhub-token'
        DOCKERHUB_USER = "ahmedmoussa"
        DOCKERHUB_PASS = credentials('dockerhub-token')
    }
    stages {
        stage('Checkout') {
            steps {
                // Use 'checkout scm' to utilize the SCM configuration defined 
                // in the Jenkins job setup (Repository URL, Branch, Credentials).
                checkout scm 
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    echo "🚀 Building cast-service image..."
                    // Note: These paths (./cast-service) are relative to the root of the checked-out repository.
                    sh 'docker build -t $DOCKERHUB_USER/cast-service:latest ./cast-service'
                    echo "🚀 Building movie-service image..."
                    sh 'docker build -t $DOCKERHUB_USER/movie-service:latest ./movie-service'
                }
            }
        }
        stage('Push Docker Images') {
            steps {
                script {
                    // Log in using the injected credentials
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
                    sh '''
                    helm upgrade --install myapp-dev ./helm/app-chart \\
                        -n dev \\
                        -f ./helm/app-chart/values-dev.yaml
                    '''
                }
            }
        }
    }
}
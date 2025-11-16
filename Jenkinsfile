pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "ahmedmoussa"
        DOCKERHUB_PASS = credentials('dockerhub-token')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ahmedmoussa92/Jenkins_datascientest.git'
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
                    helm upgrade --install myapp-dev ./helm/app-chart \
                        -n dev \
                        -f ./helm/app-chart/values-dev.yaml
                    '''
                }
            }
        }
    }
}
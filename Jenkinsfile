pipeline {
    agent any
    environment {
        DOCKERHUB_USER = "ahmedmoussa"
        // DOCKERHUB_PASS will be injected from Jenkins credentials
    }
    stages {
        stage('Checkout') {
            steps {
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
                    // Wrap Docker login and push in withCredentials
                    withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_PASS')]) {
                        sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push $DOCKERHUB_USER/cast-service:latest
                        docker push $DOCKERHUB_USER/movie-service:latest
                        '''
                    }
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
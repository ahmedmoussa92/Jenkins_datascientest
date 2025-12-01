pipeline {
    agent any

    environment {
        REGISTRY = "ahmedmoussa92"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
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
                    sh "docker build -t $REGISTRY/cast-service:$IMAGE_TAG ./cast-service"

                    echo "🚀 Building movie-service image..."
                    sh "docker build -t $REGISTRY/movie-service:$IMAGE_TAG ./movie-service"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-token',
                            usernameVariable: 'DOCKERHUB_USER',
                            passwordVariable: 'DOCKERHUB_PASS')]) {

                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'

                        sh "docker push $REGISTRY/cast-service:$IMAGE_TAG"
                        sh "docker push $REGISTRY/movie-service:$IMAGE_TAG"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install app-dev ./helm/app-chart \
                        --namespace dev --create-namespace \
                        -f ./helm/app-chart/values-dev.yaml \
                        --set cast.image=$REGISTRY/cast-service:$IMAGE_TAG \
                        --set movie.image=$REGISTRY/movie-service:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install app-qa ./helm/app-chart \
                        --namespace qa --create-namespace \
                        -f ./helm/app-chart/values-qa.yaml \
                        --set cast.image=$REGISTRY/cast-service:$IMAGE_TAG \
                        --set movie.image=$REGISTRY/movie-service:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install app-staging ./helm/app-chart \
                        --namespace staging --create-namespace \
                        -f ./helm/app-chart/values-staging.yaml \
                        --set cast.image=$REGISTRY/cast-service:$IMAGE_TAG \
                        --set movie.image=$REGISTRY/movie-service:$IMAGE_TAG
                    """
                }
            }
        }

       

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh """
                    export KUBECONFIG=$KUBECONFIG
                    helm upgrade --install app-prod ./helm/app-chart \
                        --namespace prod --create-namespace \
                        -f ./helm/app-chart/values-prod.yaml \
                        --set cast.image=$REGISTRY/cast-service:$IMAGE_TAG \
                        --set movie.image=$REGISTRY/movie-service:$IMAGE_TAG
                    """
                }
            }
        }
    }
}
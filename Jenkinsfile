pipeline {
    agent any
    environment {
        // DockerHub credentials
        DOCKERHUB_USER = "ahmedmoussa92"
        DOCKERHUB_PASS = credentials('dockerhub-token') // Secret text in Jenkins
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
                    // Log in to DockerHub using credentials
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/cast-service:latest'
                    sh 'docker push $DOCKERHUB_USER/movie-service:latest'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBECONFIG')]) {
                            sh '''
                            helm upgrade --install myapp-dev ./helm/app-chart \\
                                -n dev \\
                                -f ./helm/app-chart/values-dev.yaml
                            '''
                        }
                    }
                }
                stage('Deploy to Staging') {
                    steps {
                        withCredentials([file(credentialsId: 'kubeconfig-staging', variable: 'KUBECONFIG')]) {
                            sh '''
                            helm upgrade --install myapp-staging ./helm/app-chart \\
                                -n staging \\
                                -f ./helm/app-chart/values-staging.yaml
                            '''
                        }
                    }
                }
                stage('Deploy to Prod') {
                    steps {
                        withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
                            sh '''
                            helm upgrade --install myapp-prod ./helm/app-chart \\
                                -n prod \\
                                -f ./helm/app-chart/values-prod.yaml
                            '''
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
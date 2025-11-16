pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "ahmedmoussa92"
        DOCKERHUB_PASS = credentials('dockerhub-password-id') // add this in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/Jenkins_datascientest.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t $DOCKERHUB_USER/movie-service:latest ./movie-service'
                    sh 'docker build -t $DOCKERHUB_USER/cast-service:latest ./cast-service'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USER/movie-service:latest'
                    sh 'docker push $DOCKERHUB_USER/cast-service:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'helm upgrade --install myapp-dev ./helm/app-chart -n dev -f ./helm/app-chart/values-dev.yaml'
                }
            }
        }

        stage('Capture Jenkins Console') {
            steps {
                // This depends on your Jenkins setup; easiest way is to use "Print to PDF" plugin
                echo 'Take screenshot / export PDF after build is done'
            }
        }
    }
}
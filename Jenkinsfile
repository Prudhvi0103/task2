pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerHub')
        IMAGE_NAME = "prudhvi0103/task2-portfolio"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -e

                    export KUBECONFIG=~/.kube/config

                    echo "Checking cluster connection..."
                    kubectl get nodes

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    echo "Restarting deployment..."
                    kubectl rollout restart deployment/task2-deployment

                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/task2-deployment --timeout=180s
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline SUCCESS! Image: prudhvi0103/task2-portfolio:${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline FAILED"
        }
    }
}

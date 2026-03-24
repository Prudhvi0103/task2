pipeline {
    agent any

    environment {
        IMAGE_NAME = "prudhvi0103/task2-portfolio"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "pajeero"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'PrudhviGITHUB', url: 'https://github.com/Prudhvi0103/task2.git']])
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        // 🔥 NEW FIXED STAGE
        stage('Configure EKS Access') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                        echo "Connecting to EKS cluster..."

                        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER

                        kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
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

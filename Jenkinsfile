pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')  // Add this in Jenkins credentials
        IMAGE_NAME = "yourdockerhubusername/task2-portfolio"
        KUBE_CONFIG = credentials('kubeconfig')               // Kubernetes config credential
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'PrudhviGITHUB', url: 'https://github.com/Prudhvi0103/task2.git']])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Write kubeconfig
                    sh 'mkdir -p ~/.kube'
                    sh 'echo "$KUBE_CONFIG" > ~/.kube/config'

                    // Deploy using kubectl
                    sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                    
                    // Optional: Restart deployment to pull new image
                    sh "kubectl rollout restart deployment/task2-deployment"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful! Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}

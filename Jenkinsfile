pipeline {
    agent any

    environment {
        IMAGE_NAME = "task2-portfolio"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USER = "prudhvi0103"
        DOCKER_CREDS = "dockerHub"

        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "pajeero"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                credentialsId: 'PrudhviGITHUB',
                url: 'https://github.com/Prudhvi0103/task2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG .
                    docker tag $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "$DOCKER_CREDS",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        // 🔥 Update image in deployment.yml
        stage('Update K8s Image') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yaml
                '''
            }
        }

        // 🔥 AWS CONNECT (NO PLUGIN REQUIRED)
        stage('Configure EKS Access') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        export AWS_DEFAULT_REGION=$AWS_REGION

                        echo "Connecting to EKS..."
                        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER

                        kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/task2-deployment || true
                    kubectl get pods -o wide
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG"
        }
        failure {
            echo "❌ FAILED"
        }
    }
}

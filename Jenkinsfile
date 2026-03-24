def sendMail(stageName, status) {
    emailext(
        subject: "🚀 ${stageName} - ${status}",
        body: """
        Stage: ${stageName}
        Status: ${status}

        Job: ${env.JOB_NAME}
        Build: ${env.BUILD_NUMBER}
        URL: ${env.BUILD_URL}
        """,
        to: "viswaakumarthy1@gmail.com",   // 🔥 change this
        attachLog: true
    )
}

pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/task2-automation"

        IMAGE_NAME = "task2-portfolio"
        IMAGE_TAG = "${BUILD_NUMBER}"

        DOCKERHUB_USER = "prudhvi0103"
        DOCKER_CREDS = "dockerHub"

        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "pajeero"

        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main',
                    credentialsId: 'PrudhviGITHUB',
                    url: 'https://github.com/Prudhvi0103/task2.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                        docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
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
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Update K8s Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yaml
                    '''
                }
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/bin

                    echo "Connecting to EKS cluster..."

                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER

                    kubectl config current-context
                    kubectl get nodes
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        // 🔥 Verify + Mail
        stage('Verify Deployment') {
            steps {
                script {
                    try {
                        sh '''
                            kubectl rollout status deployment/task2-deployment
                            kubectl get pods -o wide
                            kubectl get svc
                        '''

                        sendMail("Verify Deployment", "SUCCESS")

                    } catch (err) {

                        sendMail("Verify Deployment", "FAILED")
                        throw err
                    }
                }
            }
        }
    }

    // 🔥 FINAL MAIL HERE
    post {
        success {
            emailext(
                subject: "✅ BUILD SUCCESS: ${env.JOB_NAME}",
                body: """
                🎉 Your pipeline completed SUCCESSFULLY

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """,
                to: "viswaakumarthy1@gmail.com",   // 🔥 change this
                attachLog: true
            )

            echo "✅ SUCCESS: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        }

        failure {
            emailext(
                subject: "❌ BUILD FAILED: ${env.JOB_NAME}",
                body: """
                ❌ Pipeline FAILED

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Check logs: ${env.BUILD_URL}
                """,
                to: "viswaakumarthy1@gmail.com",   // 🔥 change this
                attachLog: true
            )

            echo "❌ FAILED"
        }
    }
}

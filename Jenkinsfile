pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'rushikesh151999'
        DOCKER_CREDENTIALS = credentials('docker-hub-cred')
        BUILD_TAG = "v${BUILD_NUMBER}"
      
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Rushikamble15/k8s-server-demo.git'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        dir('frontend/todolist') {
                            sh "docker build -t ${DOCKER_USERNAME}/k8s-frontend:${BUILD_TAG} ."
                        }
                    }
                }

                stage('Build Backend') {
                    steps {
                        dir('backend') {
                            sh "docker build -t ${DOCKER_USERNAME}/k8s-backend:${BUILD_TAG} ."
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    }
                    sh "docker push ${DOCKER_USERNAME}/k8s-frontend:${BUILD_TAG}"
                    sh "docker push ${DOCKER_USERNAME}/k8s-backend:${BUILD_TAG}"
                }
            }
        }

        stage('Update Deployment with New Image Tags') {
            steps {
                script {
                    // Update frontend deployment with the new image tag
                    sh """
                        sed -i 's|rushikesh151999/k8s-frontend:.*|rushikesh151999/k8s-frontend:${BUILD_TAG}|g' k8s/frontend/deployment.yaml
                    """

                    // Update backend deployment with the new image tag
                    sh """
                        sed -i 's|rushikesh151999/k8s-backend:.*|rushikesh151999/k8s-backend:${BUILD_TAG}|g' k8s/backend/deployment.yaml
                    """
                }
            }
        }

        // stage('Deploy Monitoring Stack') {
        //     steps {
        //         script {
        //             sh """
        //                 export KUBECONFIG=/var/lib/jenkins/.kube/config
        //                 helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        //                 helm repo update
        //                 helm upgrade --install monitoring prometheus-community/kube-prometheus-stack -f k8s/helm/values.yaml -n monitoring --create-namespace
        //             """
        //         }
        //     }
        // }

        // stage('Deploy Kubernetes Dashboard') {
        //     steps {
        //         script {
        //             sh """
        //                 kubectl apply -f k8s/monitoring/kubernetes-dashboard.yaml
        //             """
        //         }
        //     }
        // }

        stage('Deploy Application') {
            steps {
                script {
                    sh """
                        kubectl apply -f k8s/mysql/
                        kubectl apply -f k8s/backend/
                        kubectl apply -f k8s/frontend/
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Deployment successful!"
                sh """
                    kubectl get pods -A
                    kubectl get services -A
                    kubectl get deployments -A
                """
            }
        }
        failure {
            script {
                echo "Pipeline failed! Getting debug information..."
                sh """
                    kubectl describe pods -l app=k8s-backend
                    kubectl describe pods -l app=k8s-frontend
                """
            }
        }
    }
}

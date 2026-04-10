pipeline {
    agent {
        label 'k8s-agent-1'
    }

    triggers {
        githubPush()
    }

    environment {
        APP_NAME    = 'my-nginx-web'
        IMAGE_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${APP_NAME}:latest ."
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat "kubectl apply -f k8s/deployment.yaml"
                    bat "kubectl apply -f k8s/service.yaml"
                    bat "kubectl apply -f k8s/ingress.yaml"
                    bat "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    bat "kubectl rollout status deployment/nginx-deployment --timeout=120s"
                    bat "kubectl get pods -l app=my-nginx"
                    bat "kubectl get svc nginx-service"
                    bat "kubectl get ingress nginx-ingress"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Access at http://my-nginx.local"
        }
        failure {
            echo "Deployment failed! Check logs for details."
        }
    }
}

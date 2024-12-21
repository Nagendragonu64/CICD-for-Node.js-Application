pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "node"
        DOCKER_REGISTRY = "nagendragonu15"
        KUBE_CONFIG_PATH = "/home/ubuntu/.kube" 
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    eval \$(minikube docker-env)  
                    docker build -t ${DOCKER_IMAGE} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                    docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}
                    docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully to Minikube!"
        }
        failure {
            echo "Deployment failed. Please check the logs."
        }
    }
}


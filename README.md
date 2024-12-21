# Node.js Application Deployment with CI/CD Pipeline

This documentation outlines the steps to deploy a Node.js application using a CI/CD pipeline with Jenkins and Kubernetes (Minikube). It also includes details on Docker integration and service exposure.

## **Project Overview**

- **Objective:** Automate the deployment of a Node.js application.
- **Technologies Used:**
  - Node.js
  - Jenkins
  - Docker
  - Kubernetes (Minikube)

## **Prerequisites**

1. **Node.js Application**:
   - I Installed a Node.js application with a valid `package.json` and required dependencies.

2. **Infrastructure Requirements:**
   - Jenkins installed and configured.
   - Docker installed and running.
   - Minikube installed and configured.

3. **Tools Installed:**
   - Docker CLI
   - Kubectl
   - Minikube
   - Jenkins plugins: Pipeline, Docker Pipeline, Kubernetes CLI.

---

## **Steps to Deploy**

### **1. Create a Dockerfile**
Please find the Dockerfile

FROM node:16
WORKDIR /usr/src/app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]


### **2. Create Kubernetes Manifests**

#### **Deployment YAML (`k8s/deployment.yaml`)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
  labels:
    app: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: 3abee25161e2 
        image: node:latest
        ports:
        - containerPort: 3000


#### **Service YAML (`k8s/service.yaml`)**

apiVersion: v1
kind: Service
metadata:
  name: nodejs-app-service
  labels:
    app: nodejs-app
spec:
  selector:
    app: nodejs-app
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
    nodePort: 31000
  type: nodeport
```

### **3. Configure Jenkins Pipeline**

#### **Jenkinsfile**

Place the following `Jenkinsfile` in your repository:

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "node:latest"
        DOCKER_REGISTRY = "nagendragonu15"
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

        stage('Deploy to Kubernetes') {
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
            echo "Application deployed successfully!"
        }
        failure {
            echo "Deployment failed. Check the logs for details."
        }
    }
}


---

### **4. Deploy the Application**

#### **1. Start Minikube**

Start the Minikube cluster:

minikube start --driver=docker


#### **2. Build and Deploy**
Run the Jenkins pipeline. It will:
1. Install dependencies.
2. Run tests.
3. Build the Docker image and deploy it to Minikube.
4. Apply Kubernetes manifests.

#### **3. Access the Application**
Expose the application using Minikube:
minikube service nodejs-app-service

This command will display the URL where the application is running.

---

## **Troubleshooting**

1. **Pod Errors:**
   - Check pod logs:
     kubectl logs <pod-name>
     kubectl logs nodejs

2. **Service Not Accessible:**
   - Verify Minikube IP:
     
     minikube ip
     minikube 3.83.231.104
     
   - Check service details:
     kubectl describe service nodejs-app-service



## **Conclusion**
This documentation provides a comprehensive guide to deploying a Node.js application using Jenkins and Kubernetes. For further customization or issues, feel free to reach out.


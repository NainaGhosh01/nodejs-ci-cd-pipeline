
# CI/CD Pipeline for Node.js Application

Overview

This repository demonstrates a Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Node.js application. The pipeline is implemented using Jenkins and integrates with Docker and Kubernetes to automate testing, building, and deploying the application.



## Features

- Automated Testing: Runs tests on every pull request to ensure code quality.
- Docker Image Build: Packages the application into a Docker image.
- Kubernetes Deployment: Deploys the Docker image to a Kubernetes cluster.
- Notifications: Sends success or failure notifications.



## Architecture
- Source Control: GitHub repository to manage the Node.js application's source code.
- CI/CD Pipeline: 
    - Runs tests using npm test.
     - Builds and pushes a Docker image to Docker Hub.
     - Deploys the image to Kubernetes using kubectl.
- Deployment Target: Kubernetes cluster.
## Prerequisites
To run this project, ensure the following are set up:

- Jenkins:
   Installed on a server or accessible through Jenkins Cloud.
   Plugins:
   - Pipeline
   - Docker Pipeline
   - Kubernetes CLI
   - Slack Notification (optional)
- Docker Hub:
  Account and credentials to push the Docker image.
- Kubernetes Cluster:
  Access to a cluster configured with kubectl.
- GitHub Repository:
   Webhooks enabled to trigger Jenkins jobs on pull requests.
## Pipeline Workflow
Stages
- Checkout Code: Clones the repository from the main branch.
- Install Dependencies: Runs npm install to install required packages.
- Run Tests: Executes npm test to verify application functionality.
- Build Docker Image: Creates a Docker image and pushes it to Docker Hub.
- Deploy to Kubernetes: 
    - Applies k8s/deployment.yaml and k8s/service.yaml to deploy the application.
- Post-Deployment: Sends notifications for success or failure.

## Jenkinsfile

Here’s the pipeline script used in Jenkins:

pipeline {
    
    agent any

    environment {
        DOCKER_IMAGE = "your-docker-repo/nodejs-app:latest"
        KUBE_CONFIG = credentials('kubeconfig-credential-id') // Kubernetes config credential
        DOCKER_CREDENTIALS = credentials('docker-credentials-id') // Docker Hub credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/nodejs-app.git'
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
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(credentialsId: 'kubeconfig-credential-id') {
                        sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
            // Notify via Slack or Email
        }
        failure {
            echo 'Deployment failed!'
            // Notify via Slack or Email
        }
    }
}

## Kubernetes Deployment Files

k8s/deployment.yaml

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
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
      - name: nodejs-app
        image: your-docker-repo/nodejs-app:latest
        ports:
        - containerPort: 3000

```
k8s/service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: nodejs-app-service
spec:
  selector:
    app: nodejs-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```
## How to use

1) Fork or Clone the Repository:

```bash
git clone https://github.com/your-username/nodejs-app-ci-cd.git
cd nodejs-app-ci-cd
```

2) Set Up Jenkins:

- Create a Jenkins pipeline job.
- Link the repository to the pipeline.
- Add necessary credentials (Docker, Kubernetes). my-project


3) Configure Kubernetes:

- Apply the YAML files in the k8s/ directory to your Kubernetes cluster:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

4) Trigger the Pipeline:
- Push changes or create a pull request to trigger the pipeline.



## Notifications
Configure notifications for deployment success or failure:

- Slack: Add Slack Notification Plugin in Jenkins and configure the webhook URL.
- Email: Add Email Notification settings in Jenkins.
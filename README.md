CI/CD Pipeline for Node.js Application with Jenkins, Docker, and Kubernetes
This project demonstrates a complete CI/CD pipeline for deploying a Node.js application using Jenkins, Docker, and Kubernetes. The pipeline automates the process of building, testing, creating Docker images, and deploying them to a Kubernetes cluster.

🛠️ Tools & Technologies Used
Jenkins: For Continuous Integration (CI) to automate the build and testing process.
Docker: For containerizing the Node.js application and deploying it to Kubernetes.
Kubernetes: For orchestrating the deployment of the Node.js application.
Slack: For sending notifications on pipeline success or failure.
GitHub: For source code version control.

📋 Prerequisites
Before you begin, ensure you have the following:
Jenkins installed with necessary plugins: Git, Pipeline, Docker, Slack Notification.
A Docker Hub account and Docker Hub credentials in Jenkins for pushing Docker images.
A Kubernetes cluster with kubectl configured to interact with the cluster.
A Slack account and webhook URL configured in Jenkins for notifications.
A GitHub repository containing the Node.js application.
🚀 How It Works
This CI/CD pipeline automatically:

Checks out the latest code from the GitHub repository.
Installs dependencies using npm install.
Runs unit tests with npm test, sending notifications if tests fail.
Builds a Docker image for the application.
Pushes the Docker image to Docker Hub.
Deploys the application to a Kubernetes cluster using a dynamic deployment YAML file.
Sends Slack notifications on pipeline success or failure.

📋 Breakdown of Pipeline Stages
1️⃣ Checkout Code
This stage pulls the latest code from the main branch of your GitHub repository:
git branch: 'main', url: 'https://github.com/your-repo/nodejs-app.git'

2️⃣ Install Dependencies
The pipeline installs the necessary dependencies using npm:
sh 'npm install'

3️⃣ Run Tests
This stage runs unit tests using npm test. If the tests fail, it sends a Slack notification and aborts the pipeline:
sh 'npm test'

4️⃣ Build Docker Image
The pipeline builds a Docker image for the application with a unique version tag based on the build number:
dockerImage = docker.build("${appRegistry}:${env.BUILD_NUMBER}")

5️⃣ Push Docker Image
This stage pushes the built Docker image to Docker Hub:
docker.withRegistry('', registryCredential) {
    dockerImage.push("${env.BUILD_NUMBER}")
    dockerImage.push("latest")
}
6️⃣ Deploy to Kubernetes
The pipeline generates a dynamic Kubernetes deployment YAML file for the application and applies it to the Kubernetes cluster. It ensures the application is deployed with two replicas:

writeFile file: 'k8s-deployment.yaml', text: """
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
      - name: nodejs-container
        image: ${appRegistry}:latest
        ports:
        - containerPort: 3000
---
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
"""
sh """
export KUBECONFIG=${kubeConfig}
kubectl apply -f k8s-deployment.yaml
"""

7️⃣ Slack Notifications
The pipeline sends notifications to a Slack channel on the success or failure of the deployment:
slackSend channel: '#deployments', color: '#36a64f', message: message, webhookToken: slackWebhook

8️⃣ Cleanup Workspace
The workspace is cleaned up after each build to ensure no unnecessary files are left behind:
cleanWs()

📝 Setup Instructions
Clone the Repository:
git clone https://github.com/your-repo/nodejs-app.git
cd nodejs-app

Set Up Jenkins:
Install Jenkins with the necessary plugins: Git, Pipeline, Docker, Slack Notification.
Create Jenkins credentials for Docker Hub, Kubernetes kubeconfig, and Slack webhook.

Configure Kubernetes Cluster:
Ensure your Kubernetes cluster is set up and accessible from Jenkins.
Install kubectl and configure it with the necessary credentials.

Set Up Slack Notifications:
Create a Slack Webhook URL and configure it in Jenkins credentials as slack-webhook-url.

Configure the Pipeline:
Set the GitHub repository URL in the Jenkinsfile.
Set up Docker Hub credentials and Kubernetes kubeconfig.
Run the Pipeline:

Trigger the pipeline manually or set up a webhook to trigger on code push.
🔧 How to Run Locally (Optional)
To test this pipeline locally with Docker and Kubernetes, follow these steps:

Build the Docker image:
docker build -t your-dockerhub-username/nodejs-app:latest .

Push to DockerHub:
docker push your-dockerhub-username/nodejs-app:latest

Deploy to Kubernetes:
kubectl apply -f k8s-deployment.yaml

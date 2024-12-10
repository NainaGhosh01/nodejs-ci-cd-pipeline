
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
- 
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

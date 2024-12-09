pipeline {
    agent any

    tools {
        nodejs "NodeJS"  // Ensure Node.js is installed in Jenkins
        dockerTool "Docker" // Docker tool configured in Jenkins
    }

    environment {
        registryCredential = 'dockerhub-creds'  // Jenkins credential ID for Docker Hub
        appRegistry = "your-dockerhub-username/nodejs-app"
        kubeConfig = credentials('kubeconfig-cred')  // K8s kubeconfig credential ID
        slackWebhook = credentials('slack-webhook-url')  // Slack webhook for notifications
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: https://github.com/NainaGhosh01/nodejs-ci-cd-pipeline'
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
            post {
                failure {
                    notifySlack("Tests failed! Pipeline aborted.")
                    error('Tests failed')
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${appRegistry}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
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
                }
            }
            post {
                success {
                    notifySlack("Deployment succeeded! Application is live.")
                }
                failure {
                    notifySlack("Deployment failed! Please check the logs.")
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after build
        }
    }
}

def notifySlack(message) {
    slackSend channel: '#deployments', color: '#36a64f', message: message, webhookToken: slackWebhook
}

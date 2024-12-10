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

pipeline {
    agent any

    environment {
        // DockerHub image reference
        DOCKER_IMAGE = "sudheerttech/trend-app:latest"

        // GitHub repo containing Kubernetes manifests
        GIT_REPO = "https://github.com/sudheerttech-eng/Trend.git"

        // AWS EKS cluster details (set these in Jenkins global env or pipeline parameters)
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "trend-cluster"
    }

    stages {
        stage('Checkout Git Repository') {
            steps {
                // Fetch deployment and service YAML files
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Login to DockerHub') {
            steps {
                // Authenticate with DockerHub using Jenkins global credentials
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                // Validate that the image exists in DockerHub
                sh "docker pull ${DOCKER_IMAGE}"
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                // Update kubeconfig to point to your EKS cluster
                sh """
                aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                // Apply Kubernetes manifests
                sh """
                echo "Applying Deployment..."
                kubectl apply -f deployment.yaml

                echo "Applying Service..."
                kubectl apply -f service.yaml
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Your app is now running on EKS."
        }
        failure {
            echo "❌ Deployment failed. Please check Jenkins logs and kubectl output."
        }
    }
}


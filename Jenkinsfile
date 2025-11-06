pipeline {
    agent any

    environment {
        REPO_URI = '785661981860.dkr.ecr.ap-south-1.amazonaws.com/devops-project-repo'
        CLUSTER_NAME = 'devops-project-cluster'
        REGION = 'ap-south-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nikkss-gif/example-voting-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the vote app Docker image from its folder
                    sh '''
                    cd vote
                    docker build -t $REPO_URI:latest .
                    '''
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REPO_URI
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                    docker push $REPO_URI:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                    aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME
                    kubectl apply -f k8s/redis-deployment.yaml
                    kubectl apply -f k8s/redis-service.yaml
                    kubectl apply -f k8s/vote-deployment.yaml
                    kubectl apply -f k8s/vote-service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
            sh 'kubectl get svc'
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}

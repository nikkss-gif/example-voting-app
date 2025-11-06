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

        stage('Deploy to EKS (Full Stack)') {
            steps {
                script {
                    sh '''
                    echo "🚀 Updating kubeconfig for cluster..."
                    aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME

                    echo "📦 Deploying Redis..."
                    kubectl apply -f k8s-specifications/redis-deployment.yaml
                    kubectl apply -f k8s-specifications/redis-service.yaml

                    echo "🗄️ Deploying PostgreSQL (DB)..."
                    kubectl apply -f k8s-specifications/db-deployment.yaml
                    kubectl apply -f k8s-specifications/db-service.yaml

                    echo "🗳️ Deploying Vote App..."
                    kubectl apply -f k8s-specifications/vote-deployment.yaml
                    kubectl apply -f k8s-specifications/vote-service.yaml

                    echo "⚙️ Deploying Worker..."
                    kubectl apply -f k8s-specifications/worker-deployment.yaml

                    echo "📊 Deploying Result App..."
                    kubectl apply -f k8s-specifications/result-deployment.yaml
                    kubectl apply -f k8s-specifications/result-service.yaml

                    echo "✅ All components applied successfully!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Full Stack Deployment Successful!"
            sh 'kubectl get svc'
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}

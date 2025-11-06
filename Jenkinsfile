pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '785661981860'
        REGION = 'ap-south-1'
        REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/devops-project-repo"
        CLUSTER_NAME = 'devops-project-cluster'
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        stage('Clean Old Deployments') {
            steps {
                script {
                    sh '''
                    echo "🧹 Cleaning previously applied Kubernetes resources..."
                    kubectl delete -f k8s-specifications --ignore-not-found=true || true
                    sleep 3
                    '''
                }
            }
        }

        stage('Build Docker Images (Parallel)') {
            parallel {
                stage('Build: Vote') {
                    steps {
                        dir('vote') {
                            sh "docker build -t ${REPO_URI}:vote-${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build: Result') {
                    steps {
                        dir('result') {
                            sh "docker build -t ${REPO_URI}:result-${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build: Worker') {
                    steps {
                        dir('worker') {
                            sh "docker build -t ${REPO_URI}:worker-${IMAGE_TAG} ."
                        }
                    }
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                    echo "🔐 Logging into AWS ECR..."
                    aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_URI}
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh '''
                    echo "📦 Pushing Docker images to ECR..."
                    docker push ${REPO_URI}:vote-${IMAGE_TAG}
                    docker push ${REPO_URI}:result-${IMAGE_TAG}
                    docker push ${REPO_URI}:worker-${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS (Full Stack)') {
            steps {
                script {
                    sh '''
                    echo "🚀 Deploying to Amazon EKS cluster: ${CLUSTER_NAME}"
                    aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER_NAME}

                    echo "📦 Applying Redis and DB..."
                    kubectl apply -f k8s-specifications/redis-deployment.yaml
                    kubectl apply -f k8s-specifications/redis-service.yaml
                    kubectl apply -f k8s-specifications/db-deployment.yaml
                    kubectl apply -f k8s-specifications/db-service.yaml

                    echo "⚙️ Deploying Vote, Worker, and Result apps..."
                    kubectl apply -f k8s-specifications/vote-deployment.yaml
                    kubectl apply -f k8s-specifications/vote-service.yaml
                    kubectl apply -f k8s-specifications/worker-deployment.yaml
                    kubectl apply -f k8s-specifications/result-deployment.yaml
                    kubectl apply -f k8s-specifications/result-service.yaml

                    echo "🕐 Waiting for pods to become ready..."
                    kubectl wait --for=condition=available deployment --all -n default --timeout=120s || true

                    echo "✅ All components applied successfully!"
                    '''
                }
            }
        }

        stage('Convert Result to LoadBalancer (Public)') {
            steps {
                script {
                    sh '''
                    echo "🌍 Converting result service to LoadBalancer..."
                    kubectl patch svc result -p '{"spec": {"type": "LoadBalancer"}}' || true

                    echo "⏳ Waiting for public IP to be assigned..."
                    for i in {1..20}; do
                      kubectl get svc result -o wide
                      sleep 5
                    done
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ ✅ Full Stack Deployment Successful!"
            sh '''
            echo "📋 Current Pod Status:"
            kubectl get pods -o wide

            echo "🌐 Current Services and Public URLs:"
            kubectl get svc -o wide
            echo ""
            echo "🎯 Access your apps:"
            echo "Vote App URL:    http://$(kubectl get svc vote-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
            echo "Result App URL:  http://$(kubectl get svc result -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
            '''
        }
        failure {
            echo "❌ Deployment failed! Check the Jenkins logs for more info."
        }
    }
}

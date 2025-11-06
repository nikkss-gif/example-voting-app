pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '785661981860'
        REGION = 'ap-south-1'
        REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/devops-project-repo"
        CLUSTER_NAME = 'devops-project-cluster'
        IMAGE_TAG = "latest" // ya "${env.GIT_COMMIT.take(8)}" for immutable tags
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Clean Old Deployments') {
            steps {
                script {
                    // ignore-not-found ensures no error if resource missing
                    sh '''
                    echo "🧹 Cleaning previously applied k8s resources (if any)..."
                    kubectl delete -f k8s-specifications --ignore-not-found=true || true
                    sleep 2
                    '''
                }
            }
        }

        stage('Build Docker Images (parallel)') {
            parallel {
                stage('Build: vote') {
                    steps {
                        dir('vote') {
                            sh "docker build -t ${REPO_URI}:vote-${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build: result') {
                    steps {
                        dir('result') {
                            sh "docker build -t ${REPO_URI}:result-${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build: worker') {
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
                    echo "🔐 Logging into ECR..."
                    aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Images to ECR') {
            steps {
                script {
                    sh '''
                    echo "📦 Pushing images to ECR..."
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
                    echo "🚀 Updating kubeconfig for ${CLUSTER_NAME}..."
                    aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER_NAME}

                    echo "📦 Applying k8s manifests from k8s-specifications..."
                    kubectl apply -f k8s-specifications/redis-deployment.yaml
                    kubectl apply -f k8s-specifications/redis-service.yaml

                    kubectl apply -f k8s-specifications/db-deployment.yaml
                    kubectl apply -f k8s-specifications/db-service.yaml

                    # Ensure image fields in deployments point to the images we pushed.
                    # Patch vote, result, worker deployments to use the newly pushed images.
                    kubectl set image deployment/vote vote=${REPO_URI}:vote-${IMAGE_TAG} --record || true
                    kubectl set image deployment/result result=${REPO_URI}:result-${IMAGE_TAG} --record || true
                    kubectl set image deployment/worker worker=${REPO_URI}:worker-${IMAGE_TAG} --record || true

                    kubectl apply -f k8s-specifications/vote-deployment.yaml
                    kubectl apply -f k8s-specifications/vote-service.yaml

                    kubectl apply -f k8s-specifications/worker-deployment.yaml

                    kubectl apply -f k8s-specifications/result-deployment.yaml
                    kubectl apply -f k8s-specifications/result-service.yaml

                    echo "✅ All components applied successfully!"
                    '''
                }
            }
        }

        stage('Optional: Convert result service -> LoadBalancer') {
            when {
                expression { return params.MAKE_RESULT_LOADBALANCER == true }
            }
            steps {
                script {
                    sh '''
                    echo "🔁 Patching result service to type=LoadBalancer..."
                    kubectl patch svc result -p '{"spec": {"type": "LoadBalancer"}}' || true
                    echo "Waiting for external IP..."
                    for i in {1..20}; do
                      kubectl get svc result -o wide
                      sleep 3
                    done
                    '''
                }
            }
        }
    }

    parameters {
        booleanParam(name: 'MAKE_RESULT_LOADBALANCER', defaultValue: false, description: 'If true, patch result service to LoadBalancer (exposes result publicly)')
    }

    post {
        success {
            echo "✅ Full Stack Deployment Successful!"
            sh 'kubectl get pods -o wide || true'
            sh 'kubectl get svc || true'
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs for details."
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKERHUB_ID = "nikhildocker976"
        IMAGE_NAME   = "vote-app"
        AWS_REGION   = "ap-south-1"
        CLUSTER_NAME = "devops-project-cluster"
    }

    stages {
        stage('🔨 Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_ID}/${IMAGE_NAME}:latest ./vote"
            }
        }

        stage('🚀 Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                    sh "docker push ${DOCKERHUB_ID}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('🚢 Deploy to Kubernetes') {
            steps {
                // Use the AWS credentials
                withAWS(credentials: 'aws-credentials', region: env.AWS_REGION) {
                    echo "Configuring kubectl for EKS cluster..."
                    // YEH SABSE ZAROORI COMMAND HAI: Cluster ka address set karo
                    sh "aws eks --region ${env.AWS_REGION} update-kubeconfig --name ${env.CLUSTER_NAME}"

                    echo "Deploying the application to EKS cluster..."
                    sh 'kubectl apply -f k8s/'
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKERHUB_ID = "nikhildocker976"
        IMAGE_NAME = "vote-app"
    }

    stages {
        stage('🔨 Build Docker Image') {
            steps {
                echo "Building the ${IMAGE_NAME} Docker image..."
                sh "docker build -t ${DOCKERHUB_ID}/${IMAGE_NAME}:latest ./vote"
            }
        }

        stage('🚀 Push to Docker Hub') {
            steps {
                echo "Pushing the ${IMAGE_NAME} image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                    sh "docker push ${DOCKERHUB_ID}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('🚢 Deploy to Kubernetes') {
            steps {
                echo "Deploying the application to EKS cluster..."
                // This command needs kubectl installed on the Jenkins server
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}

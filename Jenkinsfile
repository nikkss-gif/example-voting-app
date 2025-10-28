pipeline {
    agent any

    environment {
        // Confirm this is your correct Docker Hub ID
        DOCKERHUB_ID = "nikhildocker976"
        IMAGE_NAME = "vote-app"
    }

    stages {
        stage('🔨 Build Docker Image') {
            steps {
                echo "Building the ${IMAGE_NAME} Docker image..."
                // Tag the image with your Docker Hub ID
                sh "docker build -t ${DOCKERHUB_ID}/${IMAGE_NAME}:latest ./vote"
            }
        }

        stage('🚀 Push to Docker Hub') {
            steps {
                echo "Pushing the ${IMAGE_NAME} image to Docker Hub..."
                // Use the credentials we just saved
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // Log in to Docker Hub
                    sh "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                    // Push the image
                    sh "docker push ${DOCKERHUB_ID}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
}

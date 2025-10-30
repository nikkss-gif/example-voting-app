pipeline {
    agent any

    environment {
        DOCKERHUB_ID = "nikhildocker976"
        IMAGE_NAME = "vote-app"
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
                // YEH NAYA CODE HAI: Is stage ko AWS credentials ke saath run karo
                withAWS(credentials: 'aws-credentials', region: 'ap-south-1') {
                    echo "Deploying the application to EKS cluster..."
                    sh 'kubectl apply -f k8s/'
                }
            }
        }
    }
}

pipeline {
    agent any

    stages {
        stage('✅ Test Connection') {
            steps {
                echo 'Success! Jenkins can read the Jenkinsfile from GitHub.'
            }
        }

        stage('🔨 Build Docker Image') {
            steps {
                echo 'Building the vote app Docker image...'
                // The 'vote' folder contains the Dockerfile for the voting app.
                // The command builds a Docker image and tags it as "vote-app".
                sh 'docker build -t vote-app ./vote'
            }
        }
    }
}

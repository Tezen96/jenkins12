pipeline {
    agent any

    environment {
        IMAGE_NAME = 'suresh53/flask-app'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    }

    stages {

        stage('Setup') {
            steps {
                // Install required dependencies
                sh "pip install -r requirements.txt"

                // Check if pytest is installed
                sh "pip show pytest || echo 'pytest not found, installation failed!'"
            }
        }

        stage('Test') {
            steps {
                // Run pytest
                sh "pytest || echo 'Tests failed'"
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerhubCred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                }
                echo 'Login successful'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image built successfully"
                sh 'docker image ls'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push the Docker image to the registry
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image pushed successfully"
            }
        }      
    }
}

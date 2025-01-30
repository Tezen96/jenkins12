pipeline {
    agent any

    environment {
        IMAGE_NAME = 'suresh53/flask-app'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
        KUBECONFIG = credentials('kubeconfig-cred')
    }

    stages {
        stage('Setup') {
            steps {
                // Install pytest if not installed
                sh 'pip install pytest'
                
                // Make sure the kubeconfig file is set up correctly
                sh 'ls -la $KUBECONFIG'
                sh 'chmod 777 $KUBECONFIG'
                sh 'ls -la $KUBECONFIG'
                
                // Install dependencies from requirements.txt
                sh "pip install -r requirements.txt"
            }
        }

        stage('Test') {
            steps {
                // Run pytest using python -m to avoid PATH issues
                sh 'python -m pytest --maxfail=5 --disable-warnings -q'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerhubCred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'
                }
                echo 'Login successfully to Docker Hub'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image build successfully"
                
                // List built Docker images
                sh 'docker image ls'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push the built image to Docker Hub
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image pushed successfully"
            }
        }

        stage('Deploy to Prod') {
            steps {
                // Check the current Kubernetes context
                sh 'kubectl config current-context'
                
                // Update the Kubernetes deployment with the new image
                sh "kubectl set image deployment/flask-app flask-app=${IMAGE_TAG}"
                echo 'Deployment to Prod successful'
            }
        }
    }
}

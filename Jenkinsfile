pipeline {
    agent any

    environment {
        IMAGE_NAME = 'suresh53/flask-app'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
        KUBECONFIG = credentials('kubeconfig-cred')
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
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
                // Check Python version
                sh 'python3 --version'
                
                // Run pytest using python3 explicitly
                sh 'python3 -m pytest --maxfail=5 --disable-warnings -q'
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
        script {
            try {
                // Verify cluster connection
                sh 'kubectl config current-context'
                
                // Apply both Kubernetes configurations
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                '''
                
                // Update the deployment with the new image tag
                sh "kubectl set image deployment/flask-app flask-app=${IMAGE_TAG}"
                
                // Wait for rollout to complete
                sh 'kubectl rollout status deployment/flask-app'
                
                // Display status of all resources
                sh '''
                    echo "Checking deployment status..."
                    kubectl get deployment flask-app
                    
                    echo "Checking pod status..."
                    kubectl get pods -l app=flask-app
                    
                    echo "Checking service status..."
                    kubectl get service flask-app-service
                '''
                
            } catch (error) {
                echo "Deployment failed: ${error}"
                throw error
            }
        }
        echo 'Deployment to Prod successful'
    }
}
    }
}

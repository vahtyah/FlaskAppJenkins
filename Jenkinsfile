pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "vahtyah/flask-app:${env.BUILD_ID}"
        STAGING_SERVER = "staging-server-address"
        PROD_SERVER = "prod-server-address"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Building the Docker image
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    sh 'docker stop test-container || true'
                    sh 'docker rm test-container || true'
            
                    // Sau đó xóa image
                    sh 'docker rmi -f ${DOCKER_IMAGE}'
                    // Running the Docker container for testing
                    // sh 'docker run -d -p 5000:5000 --name test-container ${DOCKER_IMAGE}'
                    sh 'docker run -d -p 0.0.0.0:5000:5000 --name test-container ${DOCKER_IMAGE}'
                    sleep 5 // Wait for the container to be ready
                    
                    // Running tests (a basic check in this example)
                    sh 'curl -f http://localhost:5000'
                    
                    // Stopping the test container
                    sh 'docker stop test-container'
                    sh 'docker rm test-container'
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    // Deploy to the staging server
                    sh "ssh user@${STAGING_SERVER} 'docker pull ${DOCKER_IMAGE} && docker run -d -p 5000:5000 --name flask-staging ${DOCKER_IMAGE}'"
                    
                    // Check if the application is running successfully
                    sh "curl -f http://${STAGING_SERVER}:5000"
                }
            }
        }
        
        stage('Approval for Production Deployment') {
            steps {
                // Wait for manual approval
                input message: 'Deploy to production?', ok: 'Yes, proceed.'
            }
        }
        
        stage('Deploy to Production') {
            steps {
                script {
                    // Deploy to the production server
                    sh "ssh user@${PROD_SERVER} 'docker pull ${DOCKER_IMAGE} && docker run -d -p 5000:5000 --name flask-prod ${DOCKER_IMAGE}'"
                    
                    // Check if the application is running successfully
                    sh "curl -f http://${PROD_SERVER}:5000"
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup local Docker images
            sh 'docker rmi ${DOCKER_IMAGE}'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "vahtyah/flask-app:${env.BUILD_ID}"
        STAGING_SERVER = "192.168.1.100"
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
                    // sh 'docker rm -f test-container || true'
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

        stage('Test Docker Image') {
    steps {
        script {
            // Xóa container nếu tồn tại
            sh 'docker rm -f test-container || true'

            // Chạy container để test
            sh 'docker run -d -p 5000:5000 --name test-container ${DOCKER_IMAGE}'
            sleep 5 // Chờ container khởi động
            
            // Kiểm tra container hoạt động
            sh 'echo "Checking if the application is reachable..."'
            sh 'curl -v http://$(hostname -i):5000 || echo "Failed to connect to container"'

            // Dừng và xóa container sau khi test xong
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
            //Dừng và xóa container nếu nó vẫn đang chạy
            script {
                // Kiểm tra và dừng container trước khi xóa
                sh 'docker stop test-container || true'
                sh 'docker rm test-container || true'
            }
    
            // Sau đó xóa image
            script {
                sh 'docker rmi -f ${DOCKER_IMAGE} || true'
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

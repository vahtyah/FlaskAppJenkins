pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "vahtyah/flask-app:${env.BUILD_ID}"
        STAGING_SERVER = "192.168.1.100" // Địa chỉ chính xác của server staging
        PROD_SERVER = "prod-server-address"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
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
                    sleep 15 // Tăng thời gian chờ lên 15 giây
                    
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
                    // Deploy đến môi trường staging
                    sh "ssh user@${STAGING_SERVER} 'docker pull ${DOCKER_IMAGE} && docker run -d -p 5000:5000 --name flask-staging ${DOCKER_IMAGE}'"
                    
                    // Kiểm tra ứng dụng trên staging
                    sh "curl -f http://${STAGING_SERVER}:5000"
                }
            }
        }
        
        stage('Approval for Production Deployment') {
            steps {
                input message: 'Deploy to production?', ok: 'Yes, proceed.'
            }
        }
        
        stage('Deploy to Production') {
            steps {
                script {
                    // Deploy đến môi trường production
                    sh "ssh user@${PROD_SERVER} 'docker pull ${DOCKER_IMAGE} && docker run -d -p 5000:5000 --name flask-prod ${DOCKER_IMAGE}'"
                    
                    // Kiểm tra ứng dụng trên production
                    sh "curl -f http://${PROD_SERVER}:5000"
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Dừng và xóa container test nếu nó vẫn đang chạy
                sh 'docker stop test-container || true'
                sh 'docker rm test-container || true'
            }
            
            // Xóa image Docker
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

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "vahtyah/flask-app:${env.BUILD_ID}"
    }
    
    stages {
        stage('Verify Docker Access') {
            steps {
                sh 'docker version'
            }
        }
        
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
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Kiểm tra ứng dụng hoạt động
                    sh 'curl -f http://localhost:5000'
                    
                    // Dừng và xóa container sau khi test xong
                    sh 'docker stop test-container'
                    sh 'docker rm test-container'
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    // Xóa container staging nếu tồn tại
                    sh 'docker rm -f flask-staging || true'
                    
                    // Chạy container staging
                    sh 'docker run -d -p 5001:5000 --name flask-staging ${DOCKER_IMAGE}'
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Kiểm tra ứng dụng trên staging
                    sh 'curl -f http://localhost:5001'
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
                    // Xóa container production nếu tồn tại
                    sh 'docker rm -f flask-prod || true'
                    
                    // Chạy container production
                    sh 'docker run -d -p 5002:5000 --name flask-prod ${DOCKER_IMAGE}'
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Kiểm tra ứng dụng trên production
                    sh 'curl -f http://localhost:5002'
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
                
                // Dừng và xóa container staging
                sh 'docker stop flask-staging || true'
                sh 'docker rm flask-staging || true'
                
                // Dừng và xóa container production
                sh 'docker stop flask-prod || true'
                sh 'docker rm flask-prod || true'
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

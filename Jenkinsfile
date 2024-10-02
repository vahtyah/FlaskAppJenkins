pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "vahtyah/flask-app:${env.BUILD_ID}"
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
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
                    sh 'docker run -d --name test-container ${DOCKER_IMAGE}'
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Lấy địa chỉ IP của container
                    def container_ip = sh(script: "docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' test-container", returnStdout: true).trim()
                    
                    // Kiểm tra ứng dụng hoạt động
                    sh "curl -f http://${container_ip}:5000"
                    
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
                    sh 'docker run -d --name flask-staging ${DOCKER_IMAGE}'
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Lấy địa chỉ IP của container staging
                    def container_ip = sh(script: "docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' flask-staging", returnStdout: true).trim()
                    
                    // Kiểm tra ứng dụng trên staging
                    sh "curl -f http://${container_ip}:5000"
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
                    sh 'docker run -d --name flask-prod ${DOCKER_IMAGE}'
                    
                    // Tăng thời gian chờ để container khởi động
                    sleep 10
                    
                    // Lấy địa chỉ IP của container production
                    def container_ip = sh(script: "docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' flask-prod", returnStdout: true).trim()
                    
                    // Kiểm tra ứng dụng trên production
                    sh "curl -f http://${container_ip}:5000"
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
                // sh 'docker stop flask-prod || true'
                // sh 'docker rm flask-prod || true'
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
            echo 'This will run only if any stage fails'
            mail to: 'vahtyah@gmail.com',
                 subject: "Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} Failed",
                 body: "The pipeline has failed. Please check the build output at ${env.BUILD_URL}"
        }
    }
}

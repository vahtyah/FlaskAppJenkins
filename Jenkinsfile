pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                echo 'Clone...'
                git  branch: 'main', url: 'https://github.com/vahtyah/Jenkins.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                // Các bước build
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Các bước kiểm thử
            }
        }
        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to Staging...'
                // Các bước deploy đến staging

            }
        }
        stage('Deploy to Production') {
            steps {
                echo 'Deploying to Production...'
                // Các bước deploy đến production

            }
        }
    }

    post {
        always {
            echo 'This will always run after all stages'
        }
        success {
            echo 'This will run only if all stages are successful'
        }
        failure {
            echo 'This will run only if any stage fails'
            mail to: 'vahtyah@gmail.com',
                 subject: "Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER} Failed",
                 body: "The pipeline has failed. Please check the build output at ${env.BUILD_URL}"
        }
    }
}

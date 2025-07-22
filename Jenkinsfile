pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '637423323200.dkr.ecr.ap-south-1.amazonaws.com/blog-repo'
        IMAGE_VERSION = 'v1.0.0'

        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/darshanzatakiya/BLOG-IAC.git'
            }
        }

        stage('Secret Check (Safe Grep)') {
            steps {
                script {
                    def result = sh(script: """
                        grep -rE '(AKIA[0-9A-Z]{16}|aws_secret_access_key|password\\s*=\\s*)' . --exclude Jenkinsfile || true
                    """, returnStdout: true).trim()
                    
                    if (result) {
                        echo "🔒 Hardcoded secrets found:\n${result}"
                        error("Secrets detected in source code. Aborting pipeline.")
                    } else {
                        echo "✅ No hardcoded secrets found."
                    }
                }
            }
        }

        stage('Build Docker Images with Compose') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                script {
                    sh """
                        docker tag frontend ${ECR_REPO}:frontend-${IMAGE_VERSION}
                        docker tag backend ${ECR_REPO}:backend-${IMAGE_VERSION}
                        docker push ${ECR_REPO}:frontend-${IMAGE_VERSION}
                        docker push ${ECR_REPO}:backend-${IMAGE_VERSION}
                    """
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up Docker...'
            sh 'docker system prune -f'
        }
        success {
            echo '✅ Pipeline completed and images pushed to ECR.'
        }
        failure {
            echo '❌Pipeline failed.'
        }
    }
}

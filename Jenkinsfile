pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/darshanzatakiya/BLOG-IAC.git'
        IMAGE_NAME = 'blog-app'
        IMAGE_VERSION = 'v1.0.0'
        ECR_REPO = '637423323200.dkr.ecr.ap-south-1.amazonaws.com/blog-repo'
        AWS_REGION = 'ap-south-1'

        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Secret Check (Safe Grep)') {
            steps {
                script {
                    def result = sh(script: """
                        grep -rE '(AKIA[0-9A-Z]{16}|aws_secret_access_key|password\\s*=\\s*)' . --exclude Jenkinsfile || true
                    """, returnStdout: true).trim()

                    if (result) {
                        echo "❌ Hardcoded secrets found:\n${result}"
                        error("🚨 Secrets detected. Aborting pipeline.")
                    } else {
                        echo "✅ No hardcoded secrets found."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.FULL_IMAGE_TAG = "${ECR_REPO}:${IMAGE_VERSION}"
                    sh "docker build -t ${env.FULL_IMAGE_TAG} ."
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                """
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh "docker push ${env.FULL_IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up Docker resources...'
            sh 'docker system prune -f || true'
        }
        success {
            echo '✅ Pipeline completed successfully. Image pushed to ECR.'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
        }
    }
}

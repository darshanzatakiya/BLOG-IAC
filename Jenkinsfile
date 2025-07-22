pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/darshanzatakiya/BLOG-IAC.git'
        IMAGE_NAME = 'blog-app'
        IMAGE_VERSION = 'v1.0.0'

        ECR_REPO = '637423323200.dkr.ecr.ap-south-1.amazonaws.com/blog-repo'
        AWS_REGION = 'ap-south-1'

        AWS_CREDS = credentials('aws-access-key-id') // Single credential storing both key & secret
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Basic Secret Check') {
            steps {
                script {
                    def result = sh(script: """
                        grep -rE 'AKIA[0-9A-Z]{16}|aws_secret_access_key|password\\s*=\\s*' . || true
                    """, returnStdout: true).trim()

                    if (result) {
                        echo "🚨 Hardcoded secrets found:\n${result}"
                        error("Pipeline aborted due to detected secrets.")
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
                    sh "docker build -t ${FULL_IMAGE_TAG} ."
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                    aws configure set aws_access_key_id $AWS_CREDS_USR
                    aws configure set aws_secret_access_key $AWS_CREDS_PSW
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh "docker push ${FULL_IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            node {
                echo '🧹 Cleaning up Docker system...'
                sh 'docker system prune -f || true'
            }
        }

        success {
            echo '✅ Pipeline executed successfully. Docker image pushed to ECR.'
        }

        failure {
            echo '❌ Pipeline failed. Check logs for errors.'
        }
    }
}

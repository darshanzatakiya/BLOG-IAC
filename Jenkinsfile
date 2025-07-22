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

        stage('Secret Check (Basic Grep)') {
            steps {
                script {
                    def result = sh(script: """
                        grep -rE 'AKIA[0-9A-Z]{16}|aws_secret_access_key|password\\s*=\\s*' . || true
                    """, returnStdout: true).trim()
                    
                    if (result) {
                        echo "Hardcoded secrets found:\n${result}"
                        error("Secrets detected in source code. Aborting pipeline.")
                    } else {
                        echo "✅ No hardcoded secrets found."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def fullImageTag = "${ECR_REPO}:${IMAGE_VERSION}"
                    sh "docker build -t ${fullImageTag} ."
                    env.FULL_IMAGE_TAG = fullImageTag
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
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
        success {
            echo 'Pipeline executed successfully. Image pushed to ECR.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        always {
            echo 'Cleaning up Docker resources...'
            sh "docker system prune -f"
        }
    }
}


// pipeline {
//     agent any

//     environment {
//         REPO_URL = 'https://github.com/Walidbadry/Full-Stack-E-Commerce-MERN-STACK-APP.git'

//         SONARQUBE_URL = 'http://localhost:9000'  // SonarQube URL
//         SONARQUBE_TOKEN = 'your-sonarqube-token' // Replace with your SonarQube token

//         TRIVY_IMAGE = 'aquasecurity/trivy:latest'
//         DOCKER_IMAGE = 'your-docker-image:latest'
//         SCAN_REPORT = 'trivy_report.json'

//         ECR_REPO = 'your-ecr-repository-url'
//         AWS_REGION = 'your-aws-region'
//         AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
//         AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        
//        GIT_REPO = 'https://github.com/your-repository/your-repo.git'
//        GIT_BRANCH = 'main'
//        IMAGE_VERSION = 'v1.0.0' // Update with the appropriate versioning scheme
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 git branch: 'main', url: "${REPO_URL}"
//             }
//         }

//         stage('Run TruffleHog Secret Scan') {
//             steps {
//                 script {
//                     // Run TruffleHog inside Docker to scan the repository for secrets
//                     sh '''
//                     docker run --rm -it -v $(pwd):/repo trufflesecurity/trufflehog:latest github --repo https://github.com/Walidbadry/Full-Stack-E-Commerce-MERN-STACK-APP.git --json > trufflehog_output.json
//                     '''
//                 }
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 script {
//                     // Run SonarQube Scanner after TruffleHog scan
//                     withSonarQubeEnv('SonarQube') { // Jenkins will use the SonarQube server configured earlier
//                         sh '''
//                         sonar-scanner \
//                           -Dsonar.projectKey=Full-Stack-E-Commerce-MERN-STACK-APP \
//                           -Dsonar.sources=. \
//                           -Dsonar.host.url=${SONARQUBE_URL} \
//                           -Dsonar.login=${SONARQUBE_TOKEN}
//                         '''
//                     }
//                 }
//             }
//         }


//         stage('Run Dependency-Check Scan') {
//             steps {
//                 script {
//                     // Run OWASP Dependency-Check to scan for vulnerabilities
//                     sh '''
//                     dependency-check --project "MyProject" --scan . --format "HTML" --out ${REPORT_DIR}
//                     '''
//                 }
//             }
//         }

//         stage('Publish Dependency-Check Report') {
//             steps {
//                 // Archive the report as a Jenkins artifact
//                 archiveArtifacts allowEmptyArchive: true, artifacts: "${REPORT_DIR}/dependency-check-report.html"
//             }
//         }

//         stage('Fail Pipeline on Vulnerabilities') {
//             steps {
//                 script {
//                     // Check if the report contains any vulnerabilities
//                     def report = readFile("${REPORT_DIR}/dependency-check-report.html")
//                     if (report.contains("Critical") || report.contains("High")) {
//                         error "Vulnerabilities found in dependencies!"
//                     }
//                 }
//             }
//         }


//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     // Build Docker image
//                     sh 'docker build -t $DOCKER_IMAGE .'
//                 }
//             }
//         }

//         stage('Run Trivy Scan') {
//             steps {
//                 script {
//                     // Run Trivy scan on the Docker image
//                     sh '''
//                     docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/project $TRIVY_IMAGE --format json --output $SCAN_REPORT $DOCKER_IMAGE
//                     '''
//                 }
//             }
//         }

//         stage('Publish Trivy Report') {
//             steps {
//                 // Archive the Trivy scan report
//                 archiveArtifacts allowEmptyArchive: true, artifacts: "$SCAN_REPORT"
//             }
//         }

//         stage('Fail Pipeline if Vulnerabilities Found') {
//             steps {
//                 script {
//                     // Read the Trivy report and check for vulnerabilities
//                     def trivyReport = readJSON file: "$SCAN_REPORT"
//                     def criticalVulns = trivyReport.findAll { it.Vulnerability.Severity == 'CRITICAL' }
//                     def highVulns = trivyReport.findAll { it.Vulnerability.Severity == 'HIGH' }
//                     if (criticalVulns.size() > 0 || highVulns.size() > 0) {
//                         error "Critical or High vulnerabilities found!"
//                     }
//                 }
//             }
//         }


//         stage('Push Image to ECR') {
//             when {
//                 expression { 
//                     // Only run this stage if no vulnerabilities were found
//                     return currentBuild.result == null // No error means no vulnerabilities were found
//                 }
//             }
//             steps {
//                 script {
//                     // Log in to AWS ECR
//                     sh '''
//                     aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
//                     '''
                    
//                     // Tag the Docker image for ECR
//                     sh '''
//                     docker tag $DOCKER_IMAGE $ECR_REPO/$DOCKER_IMAGE
//                     '''
                    
//                     // Push the Docker image to ECR
//                     sh '''
//                     docker push $ECR_REPO/$DOCKER_IMAGE
//                     '''
//                 }
//             }
//         }


//         stage('Update Manifest Files in Git') {
//             steps {
//                 script {
//                     // Update the manifest files (e.g., Kubernetes deployment YAML or Docker Compose) with the new image version
//                     sh '''
//                     sed -i 's|image:.*|image: $ECR_REPO/$DOCKER_IMAGE:$IMAGE_VERSION|' k8s-deployment.yaml
//                     '''
//                     // Commit the changes to the repository
//                     sh '''
//                     git config --global user.email "your-email@example.com"
//                     git config --global user.name "Jenkins CI"
//                     git add k8s-deployment.yaml
//                     git commit -m "Updated image version to $IMAGE_VERSION"
//                     git push origin $GIT_BRANCH
//                     '''
//                 }
//             }
//         }

//     }

//     post {
//         always {
//             echo 'Cleaning up after build...'
//         }
//         success {
//             echo 'Build completed successfully!'
//         }
//         failure {
//             echo 'Build failed. Secret found during TruffleHog scan or SonarQube analysis failed!'
//         }
//     }
// }
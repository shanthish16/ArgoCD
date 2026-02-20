pipeline {
    agent any

    parameters {
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'test'],
            description: 'Select Branch'
        )
    }

    environment {
        AWS_REGION = "eu-north-1"
        ACCOUNT_ID = "220309168382"
        ECR_REPO = "sonar-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        PROJECT_KEY = "sonar-app"
    }

    stages {

        // ================= CHECKOUT =================
        stage('Checkout (Select Branch From Dropdown)') {
            steps {
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/shanthish16/ArgoCD.git'
            }
        }

        // ================= SONAR ANALYSIS =================
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([
                        string(credentialsId: 'sonarqube-token-K8s', variable: 'SONAR_TOKEN')
                    ]) {
                        sh """
                            mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=${PROJECT_KEY} \
                              -Dsonar.projectName=${PROJECT_KEY} \
                              -Dsonar.token=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        // ================= QUALITY GATE =================
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ================= DOCKER BUILD =================
        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building Docker image with tag ${IMAGE_TAG}"
                    
                    docker build -t ${IMAGE_URI}:${IMAGE_TAG} .
                    
                    echo "Tagging image as latest"
                    
                    docker tag ${IMAGE_URI}:${IMAGE_TAG} ${IMAGE_URI}:latest
                """
            }
        }

        // ================= ECR LOGIN =================
        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        // ================= PUSH IMAGE =================
        stage('Push Image to ECR') {
            steps {
                sh """
                    echo "Pushing build number tag ${IMAGE_TAG}"
                    docker push ${IMAGE_URI}:${IMAGE_TAG}
                    
                    echo "Pushing latest tag"
                    docker push ${IMAGE_URI}:latest
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

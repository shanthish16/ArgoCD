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
        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
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
                configFileProvider([configFile(fileId: 'nexus-settings', variable: 'MAVEN_SETTINGS')]) {
                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'sonarqube-token-K8s', variable: 'TOKEN_SONAR')]) {
                            sh """
                                mvn -B -s $MAVEN_SETTINGS clean verify sonar:sonar \
                                -Dsonar.projectKey=${PROJECT_KEY} \
                                -Dsonar.token=$TOKEN_SONAR
                            """
                        }
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
        stage('Build Docker Image (Code Built Inside Docker)') {
            steps {
                sh """
                    docker build -t ${IMAGE_URI} .
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
                    docker push ${IMAGE_URI}
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

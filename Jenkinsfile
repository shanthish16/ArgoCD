pipeline {
    agent any

    parameters {
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            defaultValue: 'main',
            branchFilter: 'origin/(.*)',
            sortMode: 'DESCENDING',
            selectedValue: 'DEFAULT',
            description: 'Select Branch to Build'
        )
    }

    environment {
        AWS_REGION = "eu-north-1"
        ACCOUNT_ID = "220309168382"
        ECR_REPO = "sonar-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout (Select Branch From Dropdown)') {
            steps {
                echo "Building branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/<username>/your-app-repo.git'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build Docker Image (Code Built Inside Docker)') {
            steps {
                echo "Building Docker Image: ${IMAGE_URI}"
                sh """
                docker build -t ${IMAGE_URI} .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                echo "Pushing Docker Image to ECR"
                sh """
                docker push ${IMAGE_URI}
                """
            }
        }
    }

    post {
        success {
            echo "Image pushed successfully to ECR"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}

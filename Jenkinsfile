pipeline {
    agent none
    environment {
        REGION="ap-southeast-1"
        FRONTEND_REPO="frontend"
        BACKEND_REPO="backend"
        REPO_FE="626557673667.dkr.ecr.ap-southeast-1.amazonaws.com/ecr-nashtech-devops-frontend"
        REPO_BE="626557673667.dkr.ecr.ap-southeast-1.amazonaws.com/ecr-nashtech-devops-backend"
        IMAGE_TAG="${BUILD_NUMBER}"
    }
    stages {
        stage('Docker Build Frontend') {
            agent any
            steps {
                withAWS(region:'ap-southeast-1',credentials:'aws dung-wru') {
                sh "pwd"
                sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_FE}"
                sh "docker build -t ${FRONTEND_REPO} src/frontend"
                sh "docker tag ${FRONTEND_REPO}:latest ${REPO_FE}:${IMAGE_TAG}"
                sh "docker push ${REPO_FE}:${IMAGE_TAG}"
                }
            }
        }
        stage('Docker Build Backend') {
            agent any
            steps {
                withAWS(region:'ap-southeast-1',credentials:'aws dung-wru') {
                sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_BE}"
                sh "docker build -t ${BACKEND_REPO} src/backend"
                sh "docker tag ${BACKEND_REPO}:latest ${REPO_BE}:${IMAGE_TAG}"
                sh "docker push ${REPO_BE}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy Frontend'){
            agent any
            steps {
                sh "export tagfe=${IMAGE_TAG}"
                sh "envsubst < frontend.yaml > fe.yaml"
                sh "kubectl apply -f fe.yaml"
                sh "kubectl get service"
            }
        }
        stage('Deploy Backend'){
            agent any
            steps {
                sh "export tagbe=${IMAGE_TAG}"
                sh "envsubst < backend.yaml > be.yaml"
                sh "kubectl apply -f be.yaml"
                sh "kubectl get service"
            }
        }
        stage('Clean up'){
            steps {
                sh "rm -rf be.yaml fe.yaml"
                sh "docker rmi ${REPO_BE}:${IMAGE_TAG}"
                sh "docker rmi ${REPO_FE}:${IMAGE_TAG}"
            }
        }
    }
}

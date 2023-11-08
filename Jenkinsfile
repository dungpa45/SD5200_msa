pipeline {
    agent none
    environment {
        REGION="ap-southeast-1"
        MY_ACR="dungacr.azurecr.io"
        FRONTEND_REPO="frontend"
        BACKEND_REPO="backend"
        REPO_FE="dungacr.azurecr.io/devops-frontend"
        REPO_BE="dungacr.azurecr.io/devops-backend"
        IMAGE_TAG="${BUILD_NUMBER}"
    }
    stages {
        stage('Login ACR'){
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                sh "docker login ${MY_ACR} -u $USERNAME -p $PASSWORD"
                }
            }
        }
        stage('Docker Build Frontend') {
            agent any
            steps {
                // withAWS(region:'ap-southeast-1',credentials:'aws dung-wru') {
                sh "pwd"
                // sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_FE}"
                sh "docker build -t ${FRONTEND_REPO} src/frontend"
                sh "docker tag ${FRONTEND_REPO}:latest ${REPO_FE}:${IMAGE_TAG}"
                sh "docker push ${REPO_FE}:${IMAGE_TAG}"
                // }
            }
        }
        stage('Docker Build Backend') {
            agent any
            steps {
                // withAWS(region:'ap-southeast-1',credentials:'aws dung-wru') {
                // sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_BE}"
                sh "docker build -t ${BACKEND_REPO} src/backend"
                sh "docker tag ${BACKEND_REPO}:latest ${REPO_BE}:${IMAGE_TAG}"
                sh "docker push ${REPO_BE}:${IMAGE_TAG}"
                // }
            }
        }
        stage('Trivy Scan Images') {
            agent any
            steps {
                // Install trivy
                // sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3'
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                // Scan all vuln levels
                sh 'mkdir -p reports'
                sh "trivy image ${REPO_FE}:${IMAGE_TAG} --format template --template '@html.tpl' --severity HIGH -o reports/frontend-scan.html"
                // Scan again and fail on CRITICAL vulns
                sh "trivy image ${REPO_BE}:${IMAGE_TAG} --format template --template '@html.tpl' --severity HIGH -o reports/backend-scan.html"
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'backend-scan.html','frontend-scan.html'
                    reportName: 'Trivy Scan',
                    reportTitles: 'Trivy Scan'
                ]
            }
        }
        stage('Deploy Frontend'){
            agent any
            steps {
                sh "export tagfe=${IMAGE_TAG};envsubst < frontend.yaml > fe.yaml"
                sh "kubectl apply -f fe.yaml"
                sh "kubectl get service"
            }
        }
        stage('Deploy Backend'){
            agent any
            steps {
                sh "export tagbe=${IMAGE_TAG};envsubst < backend.yaml > be.yaml"
                sh "kubectl apply -f be.yaml"
                sh "kubectl get service"
            }
        }
        stage('Clean up'){
            agent any
            steps {
                sh "rm -rf be.yaml fe.yaml"
                sh "docker rmi ${REPO_BE}:${IMAGE_TAG}"
                sh "docker rmi ${REPO_FE}:${IMAGE_TAG}"
            }
        }

    }
}

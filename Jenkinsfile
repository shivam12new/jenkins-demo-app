pipeline {
    agent any
    
    environment {
        ACR_NAME = 'shivamacrdemo'
        ACR_LOGIN_SERVER = 'shivamacrdemo.azurecr.io'
        IMAGE_NAME = 'jenkins-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_REPO = 'shivam12new/jenkins-demo-app'
    }
    
    stages {
        
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Docker Build') {
            steps {
                echo "Building Docker image: ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
                sh """
                    docker build -t ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Push to ACR') {
            steps {
                echo 'Pushing image to Azure Container Registry...'
                withCredentials([usernamePassword(
                    credentialsId: 'acr-credentials',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh """
                        echo ${ACR_PASS} | docker login ${ACR_LOGIN_SERVER} -u ${ACR_USER} --password-stdin
                        docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }
        
        stage('Deploy to AKS') {
            steps {
                echo 'Deploying to AKS...'
                sh """
                    sed -i 's|shivamacrdemo.azurecr.io/jenkins-demo:latest|${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml
                    kubectl apply -f deployment.yaml
                    kubectl rollout status deployment/jenkins-demo --timeout=120s
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh """
                    kubectl get pods -l app=jenkins-demo
                    kubectl get service jenkins-demo-service
                """
            }
        }
        
        stage('Auto Merge') {
            steps {
                echo 'All stages passed - Auto merging PR...'
                withCredentials([string(
                    credentialsId: 'github-token',
                    variable: 'GH_TOKEN'
                )]) {
                    sh """
                        echo "Pipeline completed successfully!"
                        echo "In production - GitHub API call would merge PR here"
                        echo "curl -X PUT -H 'Authorization: token ${GH_TOKEN}' https://api.github.com/repos/${GITHUB_REPO}/pulls/PR_NUMBER/merge"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline SUCCESS! Deployment complete!'
        }
        failure {
            echo 'Pipeline FAILED! Check logs above.'
        }
    }
}
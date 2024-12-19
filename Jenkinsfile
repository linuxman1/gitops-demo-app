pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DOCKER_REGISTRY = "linuxmanl"
        APP_NAME = "gitops-demo-app"
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git url: 'https://github.com/linuxman1/gitops-demo-app.git',
                    branch: 'main'
                
                // Debug: List files after checkout
                sh '''
                    pwd
                    ls -la
                    cat Dockerfile || echo "Dockerfile not found"
                '''
            }
        }
        
        stage('Build & Push') {
            steps {
                withCredentials([string(credentialsId: '3a0d483b-86a4-43fe-ae30-be8480dc8e6d', 
                                     variable: 'DOCKER_TOKEN')]) {
                    sh '''
                        echo "Current directory contents:"
                        ls -la
                        
                        echo "Attempting Docker login..."
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_REGISTRY" --password-stdin
                        
                        echo "Building Docker image..."
                        pwd
                        docker build -t $DOCKER_REGISTRY/$APP_NAME:$IMAGE_TAG .
                        
                        echo "Pushing Docker image..."
                        docker push $DOCKER_REGISTRY/$APP_NAME:$IMAGE_TAG
                    '''
                }
                
                withCredentials([string(credentialsId: '77937626-21b7-42a4-aa8e-a7c8eaa002cd', 
                               variable: 'GH_TOKEN')]) {
                    sh '''
                        git clone https://${GH_TOKEN}@github.com/linuxman1/gitops-demo-config.git
                        cd gitops-demo-config
                        sed -i "s|image: .*|image: ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml
                        git config user.email "jenkins@jenkins.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_TAG}"
                        git push
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            sh 'docker logout'
        }
    }
}

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
            }
        }
        
        stage('Build & Push') {
            steps {
                withCredentials([string(credentialsId: '3a0d483b-86a4-43fe-ae30-be8480dc8e6d', 
                                     variable: 'DOCKER_TOKEN')]) {
                    sh '''
                        # Login to Docker Hub
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_REGISTRY" --password-stdin
                        
                        # Build with version tag
                        docker build -t $DOCKER_REGISTRY/$APP_NAME:$IMAGE_TAG .
                        
                        # Tag as latest
                        docker tag $DOCKER_REGISTRY/$APP_NAME:$IMAGE_TAG $DOCKER_REGISTRY/$APP_NAME:latest
                        
                        # Push both tags
                        docker push $DOCKER_REGISTRY/$APP_NAME:$IMAGE_TAG
                        docker push $DOCKER_REGISTRY/$APP_NAME:latest
                        
                        # Verify images
                        docker images | grep $DOCKER_REGISTRY/$APP_NAME
                    '''
                }
                
                withCredentials([string(credentialsId: '77937626-21b7-42a4-aa8e-a7c8eaa002cd', 
                               variable: 'GH_TOKEN')]) {
                    sh '''
                        rm -rf gitops-demo-config
                        git config --global credential.helper store
                        echo "https://$GH_TOKEN:x-oauth-basic@github.com" > ~/.git-credentials
                        git clone https://github.com/linuxman1/gitops-demo-config.git
                        cd gitops-demo-config
                        sed -i "s|image: .*|image: ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml
                        git config user.email "jenkins@jenkins.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_TAG}"
                        git push origin main
                        rm -f ~/.git-credentials
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

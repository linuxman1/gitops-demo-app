pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = "linuxmanl"
        APP_NAME = "gitops-demo-app"
        GIT_CONFIG_REPO = "https://github.com/linuxman1/gitops-demo-config.git"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    def imageTag = "v${BUILD_NUMBER}"
                    
                    // Build Docker image
                    sh "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag} ."
                    
                    // Push to registry
                    withDockerRegistry([credentialsId: 'docker-cred', url: "https://${DOCKER_REGISTRY}"]) {
                        sh "docker push ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}"
                    }
                    
                    // Update version in git config repo
                    withCredentials([sshUserPrivateKey(credentialsId: 'git-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            git clone ${GIT_CONFIG_REPO} config-repo
                            cd config-repo
                            sed -i "s|\${DOCKER_IMAGE}:.*|\${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}|g" deployment.yaml
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins"
                            git add deployment.yaml
                            git commit -m "Update image to ${imageTag}"
                            git push origin main
                        """
                    }
                }
            }
        }
    }
}

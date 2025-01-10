pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:latest
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                      - name: jenkins-docker-cfg
                        mountPath: /kaniko/.docker
                  - name: git
                    image: alpine/git:latest
                    command:
                    - /bin/cat
                    tty: true
                  volumes:
                  - name: jenkins-docker-cfg
                    secret:
                      secretName: docker-credentials
                      items:
                        - key: .dockerconfigjson
                          path: config.json
            '''
        }
    }
    
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
                container('kaniko') {
                    sh """
                        /kaniko/executor \
                        --context=${env.WORKSPACE} \
                        --destination=${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} \
                        --destination=${DOCKER_REGISTRY}/${APP_NAME}:latest
                    """
                }
                
                container('git') {
                    withCredentials([string(credentialsId: '77937626-21b7-42a4-aa8e-a7c8eaa002cd', 
                                   variable: 'GH_TOKEN')]) {
                        sh """
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
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}

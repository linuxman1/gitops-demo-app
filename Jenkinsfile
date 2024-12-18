pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: gitops-demo
spec:
  containers:
  - name: nodejs
    image: node:18.17.1
    command:
    - cat
    tty: true
  - name: docker
    image: docker:24.0-dind
    securityContext:
      privileged: true
    tty: true
    volumeMounts:
      - name: docker-socket
        mountPath: /var/run/docker.sock
  volumes:
    - name: docker-socket
      emptyDir: {}
'''
        }
    }
    
    environment {
        DOCKER_REGISTRY = "linuxmanl"
        APP_NAME = "gitops-demo-app"
        GIT_CONFIG_REPO = "https://github.com/linuxman1/gitops-demo-config.git"
    }
    
    stages {
        stage('Setup') {
            steps {
                container('nodejs') {
                    sh 'node --version'
                    sh 'npm --version'
                }
                container('docker') {
                    sh 'docker --version'
                }
            }
        }
        
        stage('Build') {
            steps {
                container('nodejs') {
                    sh 'npm install'
                    sh 'npm run test || true'
                }
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                container('docker') {
                    script {
                        def imageTag = "v${BUILD_NUMBER}"
                        
                        withCredentials([string(credentialsId: 'b7e1b2de-d341-4b91-bbc5-59a463a149fc', 
                                              variable: 'DOCKER_TOKEN')]) {
                            sh """
                                echo \${DOCKER_TOKEN} | docker login -u ${DOCKER_REGISTRY} --password-stdin
                                docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag} .
                                docker push ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}
                            """
                        }
                        
                        withCredentials([string(credentialsId: 'ece93b0b-ff7c-4056-86de-c02322ad810e', 
                                              variable: 'GH_TOKEN')]) {
                            sh """
                                git clone https://\${GH_TOKEN}@github.com/linuxman1/gitops-demo-config.git config-repo
                                cd config-repo
                                sed -i "s|image:.*|image: ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}|g" deployment.yaml
                                git config user.email "jenkins@example.com"
                                git config user.name "Jenkins"
                                git commit -am "Update image to ${imageTag}"
                                git push origin main
                            """
                        }
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

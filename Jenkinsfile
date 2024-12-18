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
    - sleep
    args:
    - infinity
  - name: docker
    image: docker:24.0-dind
    securityContext:
      privileged: true
    ports:
      - containerPort: 2375
        protocol: TCP
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
      - name: DOCKER_HOST
        value: tcp://localhost:2375
    args:
      - --host=tcp://0.0.0.0:2375
      - --insecure-registry=10.0.0.0/8
      - --insecure-registry=192.168.0.0/16
      - --insecure-registry=172.16.0.0/12
      - --tls=false
  volumes:
  - name: workspace-volume
    emptyDir: {}
  - name: docker-graph-storage
    emptyDir: {}
'''
            defaultContainer 'nodejs'
        }
    }
    
    environment {
        DOCKER_REGISTRY = "linuxmanl"
        APP_NAME = "gitops-demo-app"
        GIT_CONFIG_REPO = "https://github.com/linuxman1/gitops-demo-config.git"
        DOCKER_HOST = "tcp://localhost:2375"
    }
    
    stages {
        stage('Setup') {
            steps {
                container('nodejs') {
                    sh 'node --version'
                    sh 'npm --version'
                }
                container('docker') {
                    // Wait for Docker to be ready
                    sh '''
                    while ! nc -z localhost 2375; do 
                      sleep 1
                      echo "Waiting for Docker daemon..."
                    done
                    docker info
                    '''
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

pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: docker
                    image: docker:dind
                    command: ['dockerd-entrypoint.sh']
                    args: ['--host=unix:///var/run/docker.sock', '--host=tcp://0.0.0.0:2375', '--tls=false']
                    securityContext:
                      privileged: true
                    env:
                      - name: DOCKER_TLS_CERTDIR
                        value: ''
                      - name: DOCKER_HOST
                        value: 'tcp://0.0.0.0:2375'
                  - name: jnlp
                    image: jenkins/inbound-agent:latest
                    volumeMounts:
                    - mountPath: "/home/jenkins"
                      name: "workspace-volume"
                  volumes:
                  - name: workspace-volume
                    emptyDir: {}
            '''
        }
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        DOCKER_IMAGE = "aldemaro14/java-demo-maven-ready:1.0.0"
    }
    
    stages {
        stage('Scan') {
            steps {
                withSonarQubeEnv(installationName: 'sq1') { 
                    sh './mvnw clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar'
                }
            }
            
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'ls'
                    sh 'docker build -t $DOCKER_IMAGE .'
                    sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Development Kubernetes Cluster') {
            steps {
                script {
                    // Switch Kubernetes context to the development cluster
                    sh 'kubectl config use-context dev-cluster-context'

                    // Update the Kubernetes deployment with the new image
                    sh 'kubectl set image deployment/your-deployment your-container=$DOCKER_IMAGE --namespace=your-namespace'
                }
            }
        }
    }
    post {
        always {
            // Clean up Docker credentials
            sh 'docker logout'
        }
    }
}

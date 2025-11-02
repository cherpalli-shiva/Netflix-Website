pipeline {
    agent any
    
    tools {
        jdk "jdk17"
        nodejs "node"
    }
    environment {
        DOCKER_REPO = "cherpallishiva/netflix-clone"
        IMAGE_TAG   = "${env.BUILD_NUMBER}"
    }

    stages {
        stage ('Checkout') {
            steps {
                checkout scm
            }
        }
        stage ("Install Dependencies") {
            steps {
                sh ' npm install -g yarn'
                sh 'yarn install --frozen-lockfile'
            }
        }
        stage ("Build react app") {
            steps {
                sh 'yarn build'
            }
        }
        stage ("Docker build and push") {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker build -t cherpalli/netflix-clone:jenkins .
                            docker push cherpalli/netflix-clone:jenkins
                        '''
                    }
                }
            }
        }
        stage ('Deploy to Kubernetes') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region us-east-1 --name Netflix
                    kubectl set image deployment/netflix-deployment netflix-container=cherpalli/netflix-clone:jenkins --record || true
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
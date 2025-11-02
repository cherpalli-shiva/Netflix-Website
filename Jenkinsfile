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
                            docker build -t $DOCKER_REPO:$IMAGE_TAG .
                            docker tag $DOKER_REPO:$IMAGE_TAG $DOCKER_REPO:latest
                            docker push $DOCKER_REPO:$IMAGE_TAG
                            '''
                    }
                }
            }
        }
    }
}
pipeline {
    agent any

    tools {
        jdk "jdk17"
        nodejs "node"
    }

    environment {
        DOCKER_REPO = "cherpalli/netflix-clone"
        IMAGE_TAG   = "${env.BUILD_NUMBER}"
        AWS_REGION  = "us-east-1"
        CLUSTER_NAME = "Netflix"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  npm install -g yarn
                  yarn install --frozen-lockfile
                '''
            }
        }

        stage('Build react app') {
            steps {
                sh 'yarn build'
            }
        }

        stage('Docker build and push') {
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
                            docker push $DOCKER_REPO:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                        kubectl set image deployment/netflix-deployment netflix-container=$DOCKER_REPO:$IMAGE_TAG -n default || \
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }
}

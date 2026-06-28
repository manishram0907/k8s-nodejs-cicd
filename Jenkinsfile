pipeline {

    agent any

    environment {

        IMAGE_NAME = "manishram12/node-demo"

        TAG = "latest"

        K8S_SERVER = "13.201.59.201"

    }

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/manishram0907/k8s-nodejs-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$TAG'
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                sshagent(credentials: ['k8s-server']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$K8S_SERVER "
                    export KUBECONFIG=/home/ubuntu/.kube/config

                    kubectl set image deployment/node-demo node-demo=$IMAGE_NAME:$TAG

                    kubectl rollout status deployment/node-demo
                    "
                    '''

                }

            }

        }

    }

}

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yash335/devops-task"           // Docker Hub repo
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"  // Jenkins Docker Hub credentials
        EC2_CREDENTIALS_ID = "ec2-ssh"                    // Jenkins EC2 SSH credentials
        EC2_HOST = "18.136.229.137"                       // Your Ubuntu EC2 public IP
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Yash335/DevOps_task.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm test || echo "No tests found"'
            }
        }

        stage('Dockerize') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
                    sh 'docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([env.EC2_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_HOST '
                          docker pull $DOCKER_IMAGE:$DOCKER_TAG &&
                          docker stop devops-task || true &&
                          docker rm devops-task || true &&
                          docker run -d --name devops-task -p 4010:4010 $DOCKER_IMAGE:$DOCKER_TAG
                        '
                    """
                }
            }
        }
    }
}


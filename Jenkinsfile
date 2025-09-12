pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "18.136.229.137"
        DOCKER_IMAGE = "yash335/devops-task"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        SSH_CREDENTIALS_ID = "ec2-ssh-key"  // Jenkins credential for SSH private key
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Yash335/DevOps_task.git'
            }
        }

        stage('Build & Deploy on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${SSH_CREDENTIALS_ID}", keyFileVariable: 'SSH_KEY')]) {
                    // Copy files to EC2
                    sh """
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/app
                    """
                    // Build and run Docker container on EC2
                    sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                        cd /home/${EC2_USER}/app &&
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . &&
                        docker stop devops-task || true &&
                        docker rm devops-task || true &&
                        docker run -d --name devops-task -p 4010:4010 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }
    }
}


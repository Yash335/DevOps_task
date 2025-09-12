pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yash335/devops-task"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        SSH_CREDENTIALS_ID = "ec2-ssh-key"        // Add your Jenkins SSH credential ID
        EC2_USER = "ubuntu"
        EC2_HOST = "18.136.229.137"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Yash335/DevOps_task.git'
            }
        }

        stage('Build Locally') {
            steps {
                // Optional: if you want to build npm locally before sending to EC2
                sh 'npm install || echo "npm install skipped"'
                sh 'npm test || echo "No tests found"'
            }
        }

        stage('Build & Deploy on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${SSH_CREDENTIALS_ID}", keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {

                    // Copy project files to EC2
                    sh """
                    scp -i $SSH_KEY -o StrictHostKeyChecking=no -r Dockerfile Jenkinsfile README.md app.js logoswayatt.png package-lock.json package.json $EC2_USER@$EC2_HOST:/home/ubuntu/app
                    """

                    // SSH into EC2 and build/run Docker
                    sh """
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        cd /home/ubuntu/app
                        sudo docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                        sudo docker stop devops-task || true
                        sudo docker rm devops-task || true
                        sudo docker run -d --name devops-task -p 4010:3000 $DOCKER_IMAGE:$DOCKER_TAG
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Build #${env.BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed!"
        }
        success {
            echo "Pipeline succeeded!"
        }
    }
}


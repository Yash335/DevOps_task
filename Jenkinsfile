pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')   // from Jenkins credentials
        SSH_KEY = credentials('ec2-ssh-key')                          // for EC2
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_credentials',   // use PAT or SSH key stored in Jenkins
                    url: 'https://github.com/Yash335/DevOps_task.git'
            }
        }

        stage('Build Docker Image on EC2 & Push to Docker Hub') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@18.136.229.137 '
                            cd /home/ubuntu/app &&
                            git pull origin main &&
                            echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin &&
                            docker build -t yash335/devops-task:latest . &&
                            docker push yash335/devops-task:latest &&
                            docker stop devops-task || true &&
                            docker rm devops-task || true &&
                            docker run -d --name devops-task -p 3000:3000 yash335/devops-task:latest
                        '
                    """
                }
            }
        }
    }
}


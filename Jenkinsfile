pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "yash335/devops-task"
        EC2_HOST = "ubuntu@18.136.229.137"
        APP_DIR = "/home/ubuntu/app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_credentials',
                    url: 'https://github.com/Yash335/DevOps_task.git'
            }
        }

        stage('Copy Code to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $EC2_HOST "mkdir -p $APP_DIR"
                        scp -o StrictHostKeyChecking=no -r * $EC2_HOST:$APP_DIR/
                    """
                }
            }
        }

        stage('Build, Push & Deploy on EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no $EC2_HOST '
                                cd $APP_DIR
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                                sudo docker build -t $DOCKER_USER/devops-task:latest .
                                sudo docker push $DOCKER_USER/devops-task:latest

                                sudo docker rm -f devops-task || true
                                sudo docker run -d --name devops-task -p 3000:4010 $DOCKER_USER/devops-task:latest
                            '
                        """
                    }
                }
            }
        }
    }   // <-- closes stages
}       // <-- closes pipeline


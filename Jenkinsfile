pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // DockerHub username + password/PAT
        DOCKER_HUB_REPO = "yash335/devops-task"
        APP_NAME = "devops-task"
        EC2_HOST = "ubuntu@18.136.229.137" // update with your EC2 user@IP
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Yashwant335/devops-task.git'
            }
        }

        stage('Build & Deploy on EC2') {
            steps {
                sshagent(['ec2-ssh-key']) { // Jenkins SSH key credential
                    sh '''
                    ssh -o StrictHostKeyChecking=no $EC2_HOST << 'EOF'
                        cd /home/ubuntu/app || mkdir -p /home/ubuntu/app && cd /home/ubuntu/app
                        # Pull latest code from Jenkins workspace
                        rm -rf *
                        exit
                    EOF

                    # Copy source code to EC2
                    scp -o StrictHostKeyChecking=no -r * $EC2_HOST:/home/ubuntu/app

                    # Build, push, and run container on EC2
                    ssh -o StrictHostKeyChecking=no $EC2_HOST << EOF
                        cd /home/ubuntu/app
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker build -t $DOCKER_HUB_REPO:$BUILD_NUMBER .
                        docker push $DOCKER_HUB_REPO:$BUILD_NUMBER
                        docker tag $DOCKER_HUB_REPO:$BUILD_NUMBER $DOCKER_HUB_REPO:latest
                        docker push $DOCKER_HUB_REPO:latest

                        docker stop $APP_NAME || true
                        docker rm $APP_NAME || true
                        docker run -d -p 80:3000 --name $APP_NAME $DOCKER_HUB_REPO:$BUILD_NUMBER
                    EOF
                    '''
                }
            }
        }
    }
}


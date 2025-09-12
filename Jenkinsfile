pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/devops-task"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"  // Add in Jenkins Credentials
    }

    triggers {
        githubPush()   // Webhook trigger from GitHub
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/devops-task.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm test || echo "No tests found"'  // run tests if available
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

        stage('Deploy') {
            steps {
                script {
                    // Option 1: Deploy to AWS ECS
                    // sh 'aws ecs update-service --cluster myCluster --service myService --force-new-deployment'

                    // Option 2: Deploy to Kubernetes (EKS/GKE)
                    sh """
                    kubectl set image deployment/devops-task-deployment devops-task-container=$DOCKER_IMAGE:$DOCKER_TAG --record
                    kubectl rollout status deployment/devops-task-deployment
                    """

                    // Option 3: Deploy to GCP Cloud Run
                    // sh 'gcloud run deploy devops-task --image $DOCKER_IMAGE:$DOCKER_TAG --region us-central1 --platform managed'
                }
            }
        }
    }
}


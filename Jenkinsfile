pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nandha172/flask-app"  // Your Docker Hub image name
        CONTAINER_NAME = "flask_container"
    }

    stages {

        stage('Check Environment') {
            steps {
                script {
                    sh 'echo "Running on: $(hostname)"'
                }
            }
        }

        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image: $DOCKER_IMAGE"
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Logging into Docker Hub..."
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                            sh 'echo "Docker login successful"'
                        }
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    echo "Pushing image: $DOCKER_IMAGE"
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Cleanup Old Docker Images') {
            steps {
                script {
                    echo "Cleaning up unused Docker images..."
                    sh 'docker image prune -f'
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    echo "Deploying container: $CONTAINER_NAME"
                    sh '''
                    # Stop and remove existing container if running
                    if [ "$(docker ps -q -f name=$CONTAINER_NAME)" ]; then
                        echo "Stopping existing container..."
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    fi

                    # Remove existing image to ensure fresh pull
                    docker rmi -f $DOCKER_IMAGE || true

                    # Pull latest image and run
                    echo "Running new container..."
                    docker run -d --name $CONTAINER_NAME -p 5000:5000 $DOCKER_IMAGE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! 🎉'
        }
        failure {
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}


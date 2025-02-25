pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nandha172/flask-app"  // Your Docker Hub image name
        CONTAINER_NAME = "flask_container"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh 'hostname'  // Shows where Jenkins is running
            }
        }

        stage('Clone Repository') {
            steps {
                git 'https://github.com/Nandha172/first-jenkins-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Run Flask Container') {
            steps {
                script {
                    sh '''
                    # Stop and remove existing container if running
                    if [ "$(docker ps -q -f name=$CONTAINER_NAME)" ]; then
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    fi

                    # Run the new container
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


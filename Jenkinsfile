pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nazarmalskij/prikm"
    }

    stages {
        stage('Start') {
            steps {
                echo 'Pipeline started'
            }
        }

        stage('Build & Tag Image') {
            steps {
                echo 'Building and tagging Docker image...'
                sh """
                    docker build -t $DOCKER_IMAGE:latest .
                    docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:$BUILD_NUMBER
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker images to Docker Hub...'
                withDockerRegistry([credentialsId: "dockerhub_token", url: ""]) {
                    sh "docker push $DOCKER_IMAGE:latest"
                    sh "docker push $DOCKER_IMAGE:$BUILD_NUMBER"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Docker container...'
                sh """
                    docker ps -q --filter 'ancestor=$DOCKER_IMAGE' | xargs -r docker stop
                    docker run -d -p 80:80 $DOCKER_IMAGE:latest
                """
            }
        }
    }
}

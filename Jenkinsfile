pipeline {
    agent any

    parameters {
        string(name: 'APP_PORT', defaultValue: '8080', description: 'Порт для запуску застосунку')
        booleanParam(name: 'SEND_NOTIFICATION', defaultValue: true, description: 'Надсилати повідомлення після завершення?')
    }

    environment {
        DOCKER_IMAGE = "nazarmalskij/prikm"
        TEAMS_WEBHOOK_URL = credentials('teams_webhook')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Start') {
            steps {
                echo 'Pipeline started'
                echo "Порт застосунку: ${params.APP_PORT}"
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
                    docker run -d -p ${params.APP_PORT}:80 $DOCKER_IMAGE:latest
                """
            }
        }

        stage('Notify') {
            when {
                expression { return params.SEND_NOTIFICATION }
            }
            steps {
                echo 'Sending notification to Teams...'
                sh """
                    curl -H 'Content-Type: application/json' -d '{
                        "text": "✅ Pipeline завершено успішно: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nDocker образ: $DOCKER_IMAGE:$BUILD_NUMBER"
                    }' $TEAMS_WEBHOOK_URL
                """
            }
        }
    }

    post {
        failure {
            sh """
                curl -H 'Content-Type: application/json' -d '{
                    "text": "❌ Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nПеревірте лог для деталей: ${env.BUILD_URL}"
                }' $TEAMS_WEBHOOK_URL
            """
        }
    }
}

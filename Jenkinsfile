properties {
    office365ConnectorWebhooks {
        webhooks {
            webhook {
                name('Lab_3')
                url('https://lpnu.webhook.office.com/webhookb2/3b7cf814-63b1-4da1-bcbe-cf6ff9cb00d9@7631cd62-5187-4e15-8b8e-ef653e366e7a/JenkinsCI/12d804e3988046e18a67b1d19d6c1902/824a2990-6ede-4f3c-9abb-5d0c624e0ec3/V2fjS9FKL-2EWFboJYOwMoKoclvTbd4EjA2vBAzvSOEdc1')
                startNotification(false)
                notifySuccess(true)
                notifyAborted(false)
                notifyNotBuilt(false)
                notifyUnstable(true)
                notifyFailure(true)
                notifyBackToNormal(true)
                notifyRepeatedFailure(false)
                timeout(30000)
            }
        }
    }
}

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
                sh """
                docker build -t $DOCKER_IMAGE:latest .
                docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:$BUILD_NUMBER
                """
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub_token", url: ""]) {
                    sh "docker push $DOCKER_IMAGE:latest"
                    sh "docker push $DOCKER_IMAGE:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh """
                docker ps -q --filter 'ancestor=$DOCKER_IMAGE' | xargs -r docker stop
                docker run -d -p 80:80 $DOCKER_IMAGE:latest
                """
            }
        }
    }
}

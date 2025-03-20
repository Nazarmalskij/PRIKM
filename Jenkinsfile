pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "nazarmalskij/prikm"
    }
    stages {
        stage('Start') {
            steps {
                echo 'Lab_2: started by GitHub'
            }
        }
        stage('Image build') {
            steps {
                sh "docker build -t prikm:latest ."
                sh "docker tag prikm $DOCKER_IMAGE:latest"
                sh "docker tag prikm $DOCKER_IMAGE:$BUILD_NUMBER"
            }
        }
        stage('Push to registry') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub_token", url: ""]) {
                    sh "docker push $DOCKER_IMAGE:latest"
                    sh "docker push $DOCKER_IMAGE:$BUILD_NUMBER"
                }
            }
        }
        stage('Check artifacts in Docker Hub') {
            steps {
                script {
                    def artifactCount = sh(script: "curl -s https://hub.docker.com/v2/repositories/nazarmalskij/prikm/tags/ | jq '.count'", returnStdout: true).trim()
                    echo "Current artifact count: ${artifactCount}"
                    if (artifactCount.toInteger() < 3) {
                        error("Not enough artifacts in Docker Hub! At least 3 required.")
                    }
                }
            }
        }
        stage('Update Web Page') {
            steps {
                script {
                    sh """
                    sed -i 's/BUILD_NUMBER_ENV/${BUILD_NUMBER}/' index.html
                    sed -i 's/ARTIFACT_COUNT_ENV/${artifactCount}/' index.html
                    """
                }
            }
        }
        stage('Deploy image') {
            steps {
                sh "docker ps -q --filter 'ancestor=$DOCKER_IMAGE' | xargs -r docker stop"
                sh "docker run -d -p 80:80 $DOCKER_IMAGE"
            }
        }
    }
}

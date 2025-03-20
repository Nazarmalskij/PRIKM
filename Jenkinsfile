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
        stage('Build and Push Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:latest ."
                    sh "docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:$BUILD_NUMBER"
                }
                withDockerRegistry([credentialsId: 'dockerhub_token', url: '']) {
                    sh "docker push $DOCKER_IMAGE:latest"
                    sh "docker push $DOCKER_IMAGE:$BUILD_NUMBER"
                }
            }
        }
        stage('Check Artifacts') {
            steps {
                script {
                    def artifactCount = sh(script: "curl -s https://hub.docker.com/v2/repositories/nazarmalskij/prikm/tags/ | jq '.count'", returnStdout: true).trim()
                    echo "Current artifact count: ${artifactCount}"
                    env.ARTIFACT_COUNT = artifactCount
                }
            }
        }
        stage('Update Web Page') {
            steps {
                sh "sed -i 's/BUILD_NUMBER_ENV/${BUILD_NUMBER}/g; s/ARTIFACT_COUNT_ENV/${env.ARTIFACT_COUNT}/g' index.html"
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh "docker rm -f prikm_container || true"
                    sh "docker run -d -p 80:80 --name prikm_container $DOCKER_IMAGE:latest"
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution finished!"
        }
    }
}

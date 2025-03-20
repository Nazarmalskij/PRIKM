pipeline {
    agent any
    stages {
        stage('Start') {
            steps {
                echo 'Lab_2: started by GitHub'
            }
        }
        stage('Image build') {
            steps {
                sh "docker build -t prikm:latest ."
                sh "docker tag prikm nazarmalskij/prikm:latest"
                sh "docker tag prikm nazarmalskij/prikm:$BUILD_NUMBER"
            }
        }
        stage('Push to registry') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub_token", url: ""]) {
                    sh "docker push nazarmalskij/prikm:latest"
                    sh "docker push nazarmalskij/prikm:$BUILD_NUMBER"
                }
            }
        }
        stage('Verify Image') {  // Новий стейдж
            steps {
                echo 'Verifying Docker image...'
                sh "docker images | grep nazarmalskij/prikm"
                sh "docker inspect nazarmalskij/prikm:latest"
            }
        }
        stage('Deploy image') {
            steps {
                sh "docker ps -q --filter 'ancestor=nazarmalskij/prikm' | xargs -r docker stop"
                sh "docker run -d -p 80:80 nazarmalskij/prikm"
            }
        }
    }
}

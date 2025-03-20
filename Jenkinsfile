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
                    // Перевіряємо, чи встановлено jq
                    sh "command -v jq >/dev/null 2>&1 || { echo 'jq не встановлений! Встановіть jq.'; exit 1; }"
                    
                    // Отримуємо кількість тегів у Docker Hub
                    def artifactCount = sh(script: "curl -s https://hub.docker.com/v2/repositories/nazarmalskij/prikm/tags/ | jq '.count'", returnStdout: true).trim()
                    echo "Current artifact count: ${artifactCount}"
                    
                    if (artifactCount.toInteger() < 3) {
                        error("Not enough artifacts in Docker Hub! At least 3 required.")
                    }

                    // Зберігаємо значення `artifactCount` для використання в наступних стадіях
                    env.ARTIFACT_COUNT = artifactCount
                }
            }
        }
        stage('Update Web Page') {
            steps {
                script {
                    sh """
                    sed -i 's/BUILD_NUMBER_ENV/${BUILD_NUMBER}/g' index.html
                    sed -i 's/ARTIFACT_COUNT_ENV/${env.ARTIFACT_COUNT}/g' index.html
                    """
                }
            }
        }
        stage('Deploy image') {
            steps {
                script {
                    // Зупинка запущених контейнерів (без помилки, якщо контейнерів немає)
                    sh "docker ps -q --filter 'ancestor=$DOCKER_IMAGE' | xargs -r docker stop || true"

                    // Запуск нового контейнера
                    sh "docker run -d -p 80:80 --name prikm_container $DOCKER_IMAGE"
                }
            }
        }
    }
}

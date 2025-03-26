pipeline {
    agent any
    environment {
        IMAGE_NAME = "dmytrochornnny/prikm"
    }
    stages {
        stage('Start') {
            steps {
                echo 'Lab_2: Pipeline started by GitHub trigger'
            }
        }
        stage('Cleanup old containers') {
            steps {
                sh "docker ps -q --filter ancestor=$IMAGE_NAME | xargs -r docker stop"
                sh "docker ps -aq --filter ancestor=$IMAGE_NAME | xargs -r docker rm"
            }
        }
        stage('Free port 80') {
            steps {
                sh "fuser -k 80/tcp || true"
            }
        }
        stage('Image build') {
            steps {
                sh "docker build -t prikm:latest ."
                sh "docker tag prikm $IMAGE_NAME:latest"
                sh "docker tag prikm $IMAGE_NAME:$BUILD_NUMBER"
                sh "docker tag prikm $IMAGE_NAME:stable"
            }
        }
        stage('Push to registry') {
            steps {
                withDockerRegistry([ credentialsId: "dockerhub_token", url: "" ]) {
                    sh "docker push $IMAGE_NAME:latest"
                    sh "docker push $IMAGE_NAME:$BUILD_NUMBER"
                    sh "docker push $IMAGE_NAME:stable"
                }
            }
        }
        stage('Deploy image') {
            steps {
                sh "docker run -d -p 80:80 $IMAGE_NAME:latest"
            }
        }
    }
}

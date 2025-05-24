pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-cicd-app"
        IMAGE_TAG = "latest"
        CONTAINER_NAME = "flask_cicd_container"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("${IMAGE_NAME}:${IMAGE_TAG}").inside {
                        sh 'python -m unittest discover -s .'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Якщо контейнер уже запущений, зупинити і видалити
                    sh """
                    if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    fi
                    """

                    // Запустити новий контейнер
                    sh "docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }
}


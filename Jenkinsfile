properties {
    office365ConnectorWebhooks {
        webhooks {
            webhook {
                name('Lab_3')
                url('https://lpnu.webhook.office.com/webhookb2/00caf57d-6d57-4f8c-b8bd-4ac79d0e7517@7631cd62-5187-4e15-8b8e-ef653e366e7a/JenkinsCI/f3f9c1ba0ea1460296d382953aea3929/6e03bea9-fb2c-4156-b1c3-b32f13b32c57/V29VLAtm4Wv1CG4lfT_0NevsHsQhHQAF1yi4M-NMUbohg1')
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

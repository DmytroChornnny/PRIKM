pipeline {
    agent any
    environment {
        IMAGE_NAME = "dmytrochornnny/prikm"
    }
    
    parameters {
        string(name: 'BRANCH', defaultValue: 'Lab_3', description: 'Git branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run unit tests before building?')
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: 'Environment to deploy')
    }
    
    stages {
        stage('Start') {
            steps {
                echo "Lab_2: Pipeline started by GitHub trigger"
                echo "Building branch: ${params.BRANCH}"
            }
        }
        
        stage('Checkout') {
            steps {
                script {
                    // Checkout specified branch from GitHub repository
                    git branch: params.BRANCH, url: 'https://github.com/DmytroChornnny/PRIKM.git'
                }
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

        stage('Run Tests') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                script {
                    // Running unit tests
                    echo "Running unit tests..."
                }
            }
        }

        stage('Image build') {
            steps {
                // Build Docker image
                sh "docker build -t prikm:latest ."
                // Tag the image with the proper versions
                sh "docker tag prikm $IMAGE_NAME:latest"
                sh "docker tag prikm $IMAGE_NAME:$BUILD_NUMBER"
                sh "docker tag prikm $IMAGE_NAME:stable"
            }
        }

        stage('Push to registry') {
            steps {
                withDockerRegistry([ credentialsId: "dockerhub_token", url: "" ]) {
                    // Push the Docker image to Docker Hub
                    sh "docker push $IMAGE_NAME:latest"
                    sh "docker push $IMAGE_NAME:$BUILD_NUMBER"
                    sh "docker push $IMAGE_NAME:stable"
                }
            }
        }

        stage('Deploy image') {
            steps {
                script {
                    if (params.DEPLOY_ENV == 'staging') {
                        echo "Deploying to Staging environment..."
                        // Example staging deployment commands
                        sh "docker run -d -p 80:80 --name prikm-staging $IMAGE_NAME:latest"
                    } else {
                        echo "Deploying to Production environment..."
                        // Example production deployment commands
                        sh "docker run -d -p 80:80 --name prikm-production $IMAGE_NAME:latest"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean up the workspace after pipeline execution
            cleanWs()
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "nex22183"
        IMAGE_NAME         = "samanvay-app"
        IMAGE_TAG          = "${BUILD_NUMBER}"
        CONTAINER_NAME     = "samanvay-container"
        APP_PORT           = "5000"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Pulling latest code from GitHub..."
                checkout scm
            }
        }

        stage('Run Tests') {
            steps {
                echo "Installing dependencies and running tests..."
                sh '''
                    pip3 install -r requirements.txt
                    python3 -m pytest tests/ -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh '''
                    docker build -t $DOCKERHUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG .
                    docker tag $DOCKERHUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG \
                               $DOCKERHUB_USERNAME/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo "Logging into DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Pushing image to DockerHub..."
                sh '''
                    docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container locally..."
                sh '''
                    docker rm -f $CONTAINER_NAME || true
                    docker run -d \
                        --name $CONTAINER_NAME \
                        -p $APP_PORT:5000 \
                        $DOCKERHUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    docker ps | grep $CONTAINER_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "Running health check..."
                sh '''
                    sleep 5
                    curl -f http://localhost:$APP_PORT/health
                '''
            }
        }

    }

    post {
        success {
            echo "SUCCESS: Build #${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            echo "FAILED: Build #${BUILD_NUMBER} failed. Check logs above."
        }
        always {
            echo "Pipeline finished. Cleaning up..."
            sh 'docker image prune -f'
        }
    }
}

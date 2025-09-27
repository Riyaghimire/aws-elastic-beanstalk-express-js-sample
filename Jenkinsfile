pipeline {
    agent any

    environment {
        NODE_VERSION = "16"
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        IMAGE_NAME = "riyaghimire54/${APP_NAME}"
        DOCKERHUB_CREDENTIALS = "docker-hub"   // Jenkins credential ID for Docker Hub
        SNYK_TOKEN = credentials("snyktoken")  // Jenkins secret text for Snyk token
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Install & Build') {
            steps {
                echo "Installing Node.js dependencies..."
                sh "docker run --rm -v ${WORKSPACE}:/app -w /app node:${NODE_VERSION} npm install"
                
                echo "Running tests..."
                sh "docker run --rm -v ${WORKSPACE}:/app -w /app node:${NODE_VERSION} npm test"
            }
        }

        stage('Snyk Scan') {
            steps {
                echo "Running Snyk security scan..."
                sh """
                    docker run --rm -v ${WORKSPACE}:/app -w /app node:${NODE_VERSION} /bin/sh -c \"
                        npm install -g snyk &&
                        snyk auth ${SNYK_TOKEN} &&
                        snyk test --severity-threshold=high
                    \"
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ${WORKSPACE}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs!"
        }
    }
}

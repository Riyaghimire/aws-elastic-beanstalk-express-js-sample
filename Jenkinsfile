pipeline {
    agent any

    options {
        timeout(time: 20, unit: 'MINUTES')  // Stop pipeline if it runs too long
        ansiColor('xterm')                  // Optional: colored console output
    }

    environment {
        NODE_VERSION = "16"
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        IMAGE_NAME = "riyaghimire54/${APP_NAME}"
        DOCKERHUB_CREDENTIALS = "docker-hub"   // Jenkins credentials ID for Docker Hub
        SNYK_TOKEN = credentials("snyktoken") // Jenkins secret text for Snyk token
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CloneOption', depth: 1, shallow: true]],
                    userRemoteConfigs: [[url: 'https://github.com/Riyaghimire/aws-elastic-beanstalk-express-js-sample.git',
                                         credentialsId: 'GitHub-pat']]]
                )
            }
        }

        stage('Install & Build') {
            agent {
                docker {
                    image "node:${NODE_VERSION}"
                    args '-u root:root -v $HOME/.npm:/root/.npm'  // mount npm cache
                }
            }
            steps {
                echo "Installing dependencies..."
                sh 'npm ci'  // faster, clean install
                echo "Running tests..."
                sh 'npm test'
            }
        }

        stage('Snyk Scan') {
            steps {
                echo "Running Snyk security scan..."
                sh """
                npm install -g snyk
                snyk auth ${SNYK_TOKEN}
                snyk test --severity-threshold=high
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build --pull -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
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

pipeline {
    agent any

    environment {
        NODE_VERSION = "16"
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        IMAGE_NAME = "riyaghimire54/${APP_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Install & Build') {
            agent {
                docker {
                    image "node:${NODE_VERSION}"
                    args '-u root:root' // run as root to install deps
                }
            }
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Snyk Scan') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'snyktoken', usernameVariable: 'SNYK_USER', passwordVariable: 'SNYK_TOKEN')]) {
                    sh '''
                    npm install -g snyk
                    snyk auth $SNYK_TOKEN
                    snyk test --severity-threshold=high
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:latest
                    '''
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

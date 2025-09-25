pipeline {
    agent any

    environment {
        NODE_VERSION = '16'
        DOCKER_IMAGE = 'yourdockerhubusername/app-name:latest'
    }

    stages {
        stage('Install & Test') {
            agent {
                docker { image "node:${NODE_VERSION}" }
            }
            steps {
                echo "Installing dependencies and running tests"
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Snyk Scan') {
            steps {
                echo "Running Snyk scan"
                sh 'npm install -g snyk'
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image"
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed!"
        }
    }
}

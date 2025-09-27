pipeline {
    agent any

    environment {
        NODE_VERSION = "16"
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        IMAGE_NAME = "riyaghimire54/${APP_NAME}"
        DOCKERHUB_CREDENTIALS = "docker-hub"   // Jenkins ID for Docker Hub credentials
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
                sh 'npm install'
                echo "Running tests..."
                sh 'npm test'
            }
        }

        stage('Snyk Scan') {
            steps {
                echo "Running Snyk security scan..."
                withCredentials([string(credentialsId: 'snyktoken', variable: 'SNYK_TOKEN')]) {
                    sh '''
                        npm install snyk --no-save
                        export SNYK_TOKEN=$SNYK_TOKEN
                        npx snyk test --severity-threshold=high
                    '''
                }
            }
        }

        stage('Checkout Docker Repo') {
    steps {
        echo "Cloning Docker project repo..."
        dir('Project2-Compose') {
            git url: 'https://github.com/Riyaghimire/Project2-Compose.git', branch: 'main'
        }
    }
}


stage('Build Docker Image') {
    steps {
        echo "Building Docker image from Project2-Compose..."
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -f ./Project2-Compose/jenkins/Dockerfile ./Project2-Compose"
    }
}



        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
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

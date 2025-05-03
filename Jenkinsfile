pipeline {
    agent any

    environment {
        IMAGE_NAME = "rsrs88/nodejs-app"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Python') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-pip
                    python3 --version
                '''
            }
        }

        stage('Run app.py') {
            steps {
                sh 'python3 app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'docker-cr',
                    passwordVariable: 'docker-cr-p'
                )]) {
                    sh 'echo $docker-cr-p | docker login -u $docker-cr --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker rm -f nodejs-container || true
                    docker run -d -p 3000:3000 --name nodejs-container $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}

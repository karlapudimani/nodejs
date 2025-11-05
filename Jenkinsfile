pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "karlapudi3/nodejs-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/karlapudimani/nodejs.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm -v $PWD:/app -w /app node:18-alpine sh -c "npm install && npm test"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'karlapudi3', passwordVariable: 'Password1@1608')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-18-159-209-95.eu-central-1.compute.amazonaws.com '
                    docker pull $DOCKER_IMAGE:latest &&
                    docker stop nodejs-app || true &&
                    docker rm nodejs-app || true &&
                    docker run -d -p 80:3000 --name nodejs-app $DOCKER_IMAGE:latest
                    '
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}

pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/pasindu-jayasena/Devops-Travel-Blogger.git', branch: 'main'
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker-compose build'
                }
            }
        }
        stage('Tag and Push to Docker Hub') {
            steps {
                script {
                    sh 'docker tag my-mern-app_backend yourusername/mern-backend:latest'
                    sh 'docker tag my-mern-app_frontend yourusername/mern-frontend:latest'
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push yourusername/mern-backend:latest'
                    sh 'docker push yourusername/mern-frontend:latest'
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(['ec2-ssh-credentials']) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@your-ec2-public-ip << EOF
                            docker pull yourusername/mern-backend:latest
                            docker pull yourusername/mern-frontend:latest
                            docker pull mongo:latest
                            docker stop mern-backend mern-frontend mongo || true
                            docker rm mern-backend mern-frontend mongo || true
                            docker run -d --name mongo -p 27017:27017 mongo:latest
                            docker run -d --name mern-backend -p 8000:8000 --link mongo:mongo yourusername/mern-backend:latest
                            docker run -d --name mern-frontend -p 80:5173 yourusername/mern-frontend:latest
                        EOF
                        '''
                    }
                }
            }
        }
    }
}
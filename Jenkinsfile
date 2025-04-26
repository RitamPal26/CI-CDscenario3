pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'yourdockerhubusername/java-rest-api:latest'
        AWS_EC2_IP = 'your-ec2-public-ip'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/<your-username>/<repo-name>.git', branch: 'main'
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['aws-ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${AWS_EC2_IP} '
                        docker pull $DOCKER_IMAGE
                        docker stop myapp || true
                        docker rm myapp || true
                        docker run -d -p 8080:8080 --name myapp $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }
}

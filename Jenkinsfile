pipeline {
    agent any

    environment {
        ECR_REPO = "060476966176.dkr.ecr.us-east-1.amazonaws.com/demo-app-repo"
        REGION = "us-east-1"
        EC2_USER = "ec2-user"
        EC2_IP   = "44.192.108.234"
        PEM_FILE = "/var/lib/jenkins/test99.pem"
    }

    stages {
        stage('Clone GitHub Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/avinashgangal/demo-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-app .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO
                """
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                sh """
                    docker tag demo-app:latest $ECR_REPO:latest
                    docker push $ECR_REPO:latest
                """
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no -i $PEM_FILE $EC2_USER@$EC2_IP \\
                    "docker pull $ECR_REPO:latest && \\
                     docker stop \$(docker ps -q --filter ancestor=$ECR_REPO) || true && \\
                     docker run -d -p 5000:5000 $ECR_REPO:latest"
                """
            }
        }
    }

    post {
        success {
            echo 'the new Pipeline completed successfully! Your app is live on EC2.'
        }
        failure {
            echo 'Pipeline failed. Check console output for errors.'
        }
    }
}

pipeline {
    agent any
    environment {
        ECR_REPO_URI = 'your-ecr-repo-uri'
        AWS_REGION = 'us-east-1'
        KUBECONFIG = '/path/to/kubeconfig'  // Path to your kubeconfig file
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/my-maven-app.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO_URI:latest .
                docker tag $ECR_REPO_URI:latest $ECR_REPO_URI:$(git rev-parse --short HEAD)
                '''
            }
        }
        stage('Push to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_URI
                docker push $ECR_REPO_URI:latest
                docker push $ECR_REPO_URI:$(git rev-parse --short HEAD)
                '''
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}


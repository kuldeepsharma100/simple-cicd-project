pipeline {
    agent any
    tools{
        maven 'Maven 3.9.11'
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '992382830933.dkr.ecr.us-east-1.amazonaws.com/springboot-eks-demo'
        CLUSTER_NAME = 'simple-cluster'
        S3_BUCKET = 'springboot-eks-demo-bucket'
        APP_NAME = 'springboot-eks-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kuldeepsharma100/simple-cicd-project.git'
            }
        }

        stage('Build Jar') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload Jar to S3') {
            steps {
                sh 'aws s3 cp target/*.jar s3://$S3_BUCKET/'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:latest .'
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 992382830933.dkr.ecr.$AWS_REGION.amazonaws.com
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                    sed -i "s|<ECR_REPOSITORY_URI>|$ECR_REPO|g" deployment.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                '''
            }
        }

        stage('Upload Logs to S3') {
            steps {
                sh 'aws s3 cp ${WORKSPACE}/logs s3://$S3_BUCKET/logs/ --recursive || true'
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}

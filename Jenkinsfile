pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '486414219419.dkr.ecr.us-east-1.amazonaws.com/myapp'
        CLUSTER = 'simple-eks'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                docker build -t myapp:$BUILD_NUMBER .
                docker tag myapp:$BUILD_NUMBER $ECR_REPO:$BUILD_NUMBER
                docker push $ECR_REPO:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --name $CLUSTER --region $AWS_REGION
                kubectl set image deployment/myapp myapp=$ECR_REPO:$BUILD_NUMBER --record
                kubectl rollout status deployment/myapp --timeout=120s
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                LOAD_BALANCER=$(kubectl get svc myapp-svc -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")
                echo "App URL: http://$LOAD_BALANCER"
                curl -f http://$LOAD_BALANCER || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployed successfully! BUILD: ${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Deployment failed. Rolling back..."
            sh '''
            aws eks update-kubeconfig --name $CLUSTER --region $AWS_REGION
            kubectl rollout undo deployment/myapp
            '''
        }
    }
}


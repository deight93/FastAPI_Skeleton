pipeline {
    agent any

    environment {
       NAME = "fastapi-test"
       ECR_REPO = "974455589565.dkr.ecr.ap-northeast-1.amazonaws.com"
       AWS_CREDENTIALS = "ecr-token"
       REGION = "ap-northeast-1"
       TARGET_HOST = "fast@54.95.76.21"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_access_token',
                    url: 'https://github.com/deight93/FastAPI_SQLAlchemy.git'
            }
        }
        
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/deight93/FastAPI_SQLAlchemy'
            }
        }
        
        stage('Build') {
            steps {
                sh "docker build -t ${NAME} ."
                sh "docker tag ${NAME}:latest ${ECR_REPO}/${NAME}:latest"
            }
        }
        
        stage('ECR Upload'){
            steps {
                script{
                    docker.withRegistry("https://${ECR_REPO}", "ecr:${REGION}:${AWS_CREDENTIALS}") {
                      docker.image("${ECR_REPO}/${NAME}:latest").push()
                    }
                }
            }
        }

        stage('fast api deploy') {
            steps {        
                sshagent (credentials: ['jenkins-ssh']) {
                sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "docker stop fastapi"
                    ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "docker rm fastapi"
                    ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "docker run -d --name fastapi --network docker-net -p 8000:8000 974455589565.dkr.ecr.ap-northeast-1.amazonaws.com/fastapi-test:latest"
                """
                }
            }
        }
    }
}
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install boto3'
            }
        }

        stage('Deploy to S3') {
            steps {
                sh 'python3 deploy.py'
            }
        }
    }
}

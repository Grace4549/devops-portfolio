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

        stage('Setup Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                source venv/bin/activate
                pip install --upgrade pip
                pip install boto3
                '''
            }
        }

        stage('Deploy to S3') {
            steps {
                sh '''
                source venv/bin/activate
                python3 deploy.py
                '''
            }
        }
    }
}

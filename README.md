# Automating Website Deployment to AWS S3 with Jenkins CI/CD

One of the core principles of DevOps is that deployment should never be a manual process. Every time you push code, the pipeline should take over; test, build, and deploy without you lifting a finger. This project is my practical implementation of that principle.

I built a Jenkins CI/CD pipeline that automatically deploys a static website to AWS S3 whenever code is pushed, using Python and Boto3 for the deployment logic and Jenkins credentials management to handle AWS authentication securely.

---

## Architecture

```
Developer pushes code
  ↓
Jenkins detects change (SCM Checkout)
  ↓
Python virtual environment set up
  ↓
Boto3 deploys files to AWS S3
  ↓
Website live on S3
```

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| CI/CD Server | Jenkins |
| Deployment Script | Python, Boto3 |
| Cloud Storage | AWS S3 |
| Credentials Management | Jenkins Credentials Store |
| Virtual Environment | Python venv |
| Version Control | Git, GitHub |
| Frontend | HTML, CSS, JavaScript |

---

## The Jenkins Pipeline

The pipeline has three stages:

```groovy
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
                . venv/bin/activate
                pip install --upgrade pip
                pip install boto3
                '''
            }
        }
        stage('Deploy to S3') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    . venv/bin/activate
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    python3 deploy.py
                    '''
                }
            }
        }
    }
}
```

### Why three stages?

**Checkout Code** — Jenkins pulls the latest code from GitHub. This ensures every deployment is based on what is actually in the repository, not what is on someone's local machine.

**Setup Virtual Environment** — Instead of installing dependencies globally on the Jenkins server, a Python virtual environment is created fresh for each build. This keeps the environment clean and reproducible.

**Deploy to S3** — AWS credentials are injected securely from Jenkins' credentials store using `withCredentials`. This means the actual keys never appear in the code or logs. A critical security practice.

---

## The Deployment Script

```python
import boto3
import os

bucket_name = "grace-professional-portfolio-2026"
s3 = boto3.client("s3")

files = ["index.html", "style.css", "script.js"]

for file in files:
    s3.upload_file(
        file,
        bucket_name,
        file,
        ExtraArgs={
            "ContentType": "text/html" if file.endswith(".html")
            else "text/css" if file.endswith(".css")
            else "application/javascript"
        }
    )

print("Website deployed successfully")
```

Setting the correct `ContentType` for each file type is important. Without it, S3 serves all files as plain text and the browser cannot render them correctly. This is a detail that is easy to miss and causes confusing behaviour when first encountered.

---

## Security Approach

AWS credentials are never hardcoded in the code or the Jenkinsfile. They are stored in Jenkins' built-in credentials store and injected as environment variables at runtime using `withCredentials`. This means:

- Credentials never appear in source code
- Credentials never appear in build logs
- Rotating credentials only requires updating the Jenkins store, not the code

This is the correct way to handle secrets in a CI/CD pipeline and mirrors how it is done in professional engineering teams.

---

## Challenges I Ran Into

**AWS credentials not being recognised** — Initially passed credentials incorrectly in the pipeline. The fix was using Jenkins' `withCredentials` block properly and ensuring the credential IDs matched exactly what was stored in Jenkins. A small mismatch in naming caused the pipeline to fail silently.

**Wrong content types on S3** — Files uploaded without explicit content types were served incorrectly by the browser. Adding the `ContentType` logic to the deploy script fixed this and was an important lesson about how S3 static hosting works.

**Virtual environment activation in shell steps** — Jenkins runs each `sh` block in a new shell session, meaning the virtual environment activated in one step does not carry over to the next. Activating it within the same shell block solved this.

---

## What I Learned

- How to structure a multi-stage Jenkins pipeline
- How to manage AWS credentials securely in Jenkins using the credentials store
- How Python virtual environments work in CI/CD contexts
- How Boto3 interacts with AWS S3 for programmatic file uploads
- Why content types matter when serving files from S3
- The importance of keeping secrets out of source code

---

## What Is Coming Next

- Add a build stage with automated testing before deployment
- Slack or email notifications on pipeline success or failure
- Rollback mechanism if deployment fails
- GitHub Actions version of the same pipeline for comparison
- CloudFront CDN integration for the S3 hosted site

---

## Author

**Grace Kihonge** — Cloud & DevOps Engineer based in Nairobi, Kenya.
Transitioning from IT Customer Support into Cloud & DevOps, building real infrastructure and documenting everything publicly.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/grace-kihonge-1354a9310/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Grace4549)

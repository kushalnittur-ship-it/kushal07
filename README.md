Automate Code Scanning in CI/CD – Step by Step

Automating code scanning in CI/CD means every code change is automatically checked for:

Security vulnerabilities
Code quality issues
Secrets/password leaks
Dependency vulnerabilities
Container image risks
2. Architecture Flow
Developer Pushes Code
        ↓
GitHub / GitLab / Bitbucket
        ↓
CI/CD Pipeline Triggered
        ↓
Code Checkout
        ↓
Run SAST Scan
        ↓
Run Dependency Scan
        ↓
Run Secret Scan
        ↓
Build Docker Image
        ↓
Run Container Scan
        ↓
Deploy to Test Environment
        ↓
Run DAST Scan
        ↓
 
 Step 1 — Create Git Repository

Create repository in:

GitHub
GitLab

Example repo structure:

project/
 ├── app.py
 ├── requirements.txt
 ├── Dockerfile
 ├── Jenkinsfile
 └── .github/workflows/

 
 Step 2 — Install Required Tools
Install Docker

Docker

Ubuntu:

sudo apt update
sudo apt install docker.io -y
Install Jenkins

Jenkins

sudo apt install openjdk-17-jdk -y


Step 3 — Install Code Scanning Tools
A. Semgrep (SAST)
pip install semgrep

Test:

semgrep --version

Run scan:

semgrep scan .
B. Trivy (Dependency + Image Scan)

Trivy

Install:

sudo apt install wget apt-transport-https gnupg lsb-release -y

Run:

trivy fs .

Docker image scan:

trivy image nginx:latest
C. Gitleaks (Secrets Scan)

Gitleaks

Run:

gitleaks detect .
D. SonarQube

Install using Docker:

docker run -d --name sonarqube \
-p 9000:9000 sonarqube:lts

Access:

http://localhost:9000

Default login:

admin/admin


Step 4 — Create CI/CD Pipeline
Example Using Jenkins

Create Jenkinsfile

pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/example/project.git'
            }
        }

        stage('Semgrep Scan') {
            steps {
                sh 'semgrep scan .'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Secrets Scan') {
            steps {
                sh 'gitleaks detect .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-app .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image demo-app'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
            }
        }
    }
}


Step 5 — GitHub Actions Automation

Create:

.github/workflows/security.yml

Example:

name: Security Scan

on:
  push:
    branches:
      - main

jobs:
  scan:
    runs-on: ubuntu-latest

    steps:

    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Install Semgrep
      run: pip install semgrep

    - name: Run Semgrep
      run: semgrep scan .

    - name: Install Trivy
      run: |
        sudo apt-get install wget -y
        wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
        sudo dpkg -i trivy_0.50.1_Linux-64bit.deb

    - name: Run Trivy
      run: trivy fs .

    - name: Run Gitleaks
      uses: gitleaks/gitleaks-action@v2


Step 6 — Configure Failure Conditions

Pipeline should fail if:

Critical vulnerabilities found
Secrets detected
Quality gate failed

Example:

semgrep scan . --error
trivy fs --exit-code 1 --severity HIGH,CRITICAL .


Step 7 — Generate Reports

Generate JSON reports:

semgrep scan --json > semgrep-report.json
trivy fs --format json -o trivy-report.json .


Step 8 — Email / Slack Notifications

Example Jenkins:

post {
    failure {
        mail to: 'team@example.com',
        subject: 'Build Failed',
        body: 'Security vulnerabilities found'
    }
}


Step 9 — Deploy Only if Scan Passes

Flow:

Code Push
   ↓
Scan Code
   ↓
No Critical Issues?
   ↓
YES → Deploy
NO  → Stop Pipeline. 

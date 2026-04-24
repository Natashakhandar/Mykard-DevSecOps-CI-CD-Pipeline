# Mykard DevSecOps CI/CD Pipeline

## Overview

This project demonstrates an end-to-end DevSecOps CI/CD pipeline integrating code quality analysis, security scanning, and automated deployment. The pipeline ensures secure, reliable, and continuous delivery of applications using industry-standard tools.

---

## Tech Stack

* GitHub – Source code management
* Jenkins – CI/CD pipeline automation
* SonarQube – Static code analysis
* OWASP Dependency Check – Dependency vulnerability scanning
* Trivy – File system security scanning
* Docker and Docker Compose – Application deployment
* AWS EC2 – Hosting environment

---

## Architecture

GitHub → Jenkins → SonarQube → OWASP Dependency Check → Trivy → Docker Deployment

---

## Prerequisites

* Ubuntu (EC2 instance recommended)
* Minimum 2 GB RAM (4 GB recommended for SonarQube)
* Open ports: 22, 8080, 9000, 5000, 3000

---

## Step 1: Install Dependencies

```bash
sudo apt update
sudo apt install -y docker.io docker-compose openjdk-11-jdk git
```

---

## Step 2: Install Jenkins

```bash
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Get Jenkins initial password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Access Jenkins:
http://<your-ec2-ip>:8080

---

## Step 3: Start Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Add Jenkins to Docker group:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## Step 4: SonarQube Setup

Set required kernel parameter:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Run SonarQube container:

```bash
docker run -d \
--name sonarqube-server \
-p 9000:9000 \
sonarqube:lts
```

Access SonarQube:
http://<your-ec2-ip>:9000

Default login:

* Username: admin
* Password: admin

---

## Step 5: Install Trivy

```bash
sudo apt install -y wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb stable main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install -y trivy
```

---

## Step 6: Jenkins Configuration

### Install Plugins

* SonarQube Scanner
* Sonar Quality Gates
* OWASP Dependency Check
* Docker
* Git

---

### Add SonarQube Token

1. SonarQube → My Account → Security → Generate Token
2. Jenkins → Manage Jenkins → Credentials → Add Credentials

```
Kind: Secret Text  
ID: sonar  
Value: <your-sonar-token>
```

---

### Configure SonarQube Server

Manage Jenkins → Configure System

```
Name: sonar  
Server URL: http://localhost:9000  
Credential: sonar
```

---

### Configure OWASP Tool

Manage Jenkins → Global Tool Configuration

```
Name: dc  
Install automatically
```

---

## Step 7: GitHub Setup

If repository is private, generate token:

```text
https://github.com/settings/tokens
```

Add in Jenkins:

```
Kind: Username with password  
Username: your-username  
Password: your-token  
ID: github-creds
```

---

## Step 8: Jenkins Pipeline

```groovy
pipeline {
    agent any  

    environment {
        SONAR_HOME = tool "sonar"
    }

    stages {

        stage('Clone Code') {
            steps {
                deleteDir()
                git branch: 'main',
                    url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectKey=mykard -Dsonar.projectName=mykard"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan .',
                    odcInstallation: 'dc'
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-report.txt .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker-compose down || true
                docker-compose up -d --build
                '''
            }
        }
    }
}
```

---

## Step 9: Run Pipeline

* Go to Jenkins Dashboard
* Click "Build Now"
* Monitor stages

---

## Verification Commands

Check running containers:

```bash
docker ps
```

Test backend:

```bash
curl http://localhost:5000
```

Test frontend:

```bash
curl http://localhost:3000
```

---

## Screenshots Section

Add screenshots for:

* Jenkins pipeline stages
* SonarQube dashboard
* OWASP report
* Trivy scan output
* Docker containers
* Application running in browser

---

## Key Features

* Automated CI/CD pipeline
* Integrated security scanning
* Early vulnerability detection
* Containerized deployment
* Cloud-based infrastructure

---

## Future Improvements

* Slack notifications
* Kubernetes deployment
* Monitoring with Grafana
* Automated vulnerability tracking

---

## Conclusion

This project demonstrates how to integrate security into CI/CD pipelines, ensuring applications are secure, reliable, and production-ready before deployment.

# DevOps Project Report: Automated CI/CD Pipeline for a 2-Tier Flask Application on AWS

**Author:** Rajakumar Kalashetti  
**Date:** May 19, 2026

---

# Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Step 1: AWS EC2 Instance Preparation](#3-step-1-aws-ec2-instance-preparation)
4. [Step 2: Install Dependencies on EC2](#4-step-2-install-dependencies-on-ec2)
5. [Step 3: Jenkins Installation and Setup](#5-step-3-jenkins-installation-and-setup)
6. [Step 4: GitHub Repository Configuration](#6-step-4-github-repository-configuration)
  - [Dockerfile](#dockerfile)
  - [docker-compose.yml](#docker-composeyml)
  - [Jenkinsfile](#jenkinsfile)
7. [Step 5: Jenkins Pipeline Creation and Execution](#7-step-5-jenkins-pipeline-creation-and-execution)
8. [Conclusion](#8-conclusion)
9. [Infrastructure Diagram](#9-infrastructure-diagram)
10. [Workflow Diagram](#10-workflow-diagram)

---

# 1. Project Overview

This document outlines the step-by-step process for deploying a 2-tier web application (Flask + MySQL) on an AWS EC2 instance. The deployment is containerized using Docker and Docker Compose. A complete CI/CD pipeline is established using Jenkins to automate the build and deployment process whenever new code is pushed to a GitHub repository.

---

# 2. Architecture Diagram

```text
+-----------------+      +----------------------+      +-----------------------------+
|   Developer     |----->|     GitHub Repo      |----->|        Jenkins Server       |
| (pushes code)   |      | (Source Code Mgmt)   |      |       (AWS EC2)             |
+-----------------+      +----------------------+      |                             |
                                                       | 1. Clone Repository         |
                                                       | 2. Build Docker Image       |
                                                       | 3. Run Docker Compose       |
                                                       +--------------+--------------+
                                                                      |
                                                                      | Deploys
                                                                      v
                                                       +-----------------------------+
                                                       |      Application Server     |
                                                       |        (AWS EC2)            |
                                                       |                             |
                                                       | +-------------------------+ |
                                                       | | Docker Container Flask | |
                                                       | +-------------------------+ |
                                                       |              |              |
                                                       |              v              |
                                                       | +-------------------------+ |
                                                       | | Docker Container MySQL | |
                                                       | +-------------------------+ |
                                                       +-----------------------------+
```

---

# 3. Step 1: AWS EC2 Instance Preparation

## Launch EC2 Instance

- Open AWS EC2 Console
- Launch Ubuntu 22.04 LTS Instance
- Select `t2.micro`
- Create Key Pair

---

## Configure Security Group

| Type | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| Flask App | 5000 |
| Jenkins | 8080 |

---

## Connect to EC2

```bash
ssh -i key.pem ubuntu@<ec2-public-ip>
```

---

# 4. Step 2: Install Dependencies on EC2

## Update Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Install Docker, Git, Compose

```bash
sudo apt install git docker.io docker-compose-v2 -y
```

---

## Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Add User to Docker Group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 5. Step 3: Jenkins Installation and Setup

## Install Java

```bash
sudo apt install openjdk-17-jdk -y
```

---

## Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

```bash
sudo apt update
sudo apt install jenkins -y
```

---

## Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Get Jenkins Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

```text
http://<ec2-public-ip>:8080
```

---

## Give Docker Permission to Jenkins

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

# 6. Step 4: GitHub Repository Configuration

Project files:

```text
app.py
Dockerfile
docker-compose.yml
Jenkinsfile
requirements.txt
templates/
static/
```

---

# Dockerfile

```dockerfile
FROM python:3.9

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir flask flask-mysqldb bcrypt mysqlclient

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# docker-compose.yml

```yaml
version: '3.8'

services:

  flask-app:
    build: .
    container_name: flask-auth-app

    ports:
      - "5000:5000"

    depends_on:
      - mysql-db

    environment:
      MYSQL_HOST: mysql-db
      MYSQL_USER: root
      MYSQL_PASSWORD: root123
      MYSQL_DB: authapp

    restart: always

  mysql-db:
    image: mysql:8

    container_name: mysql-db

    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: authapp

    ports:
      - "3306:3306"

    restart: always

    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

---

# Jenkinsfile

```groovy
pipeline {

    agent any

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/rajakumarck28/Automated-CI-CD-Pipeline-for-a-2-Tier-Flask-Application-on-AWS.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-auth-app .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker rm -f flask-auth-app || true'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 5000:5000 --name flask-auth-app flask-auth-app'
            }
        }

    }
}
```

---

# 7. Step 5: Jenkins Pipeline Creation and Execution

## Create Jenkins Pipeline

- New Item
- Pipeline
- Pipeline Script from SCM
- Git
- Add GitHub Repository URL

---

## Build Pipeline

Click:

```text
Build Now
```

---

## Verify Containers

```bash
docker ps
```

---

## Access Application

```text
http://<ec2-public-ip>:5000
```

---

# 8. Conclusion

The CI/CD pipeline is fully automated using Jenkins and Docker. Whenever code is pushed to GitHub, Jenkins automatically builds and deploys the updated Flask application on AWS EC2.

---

# 9. Infrastructure Diagram

```text
AWS EC2
│
├── Jenkins
├── Docker
│   ├── Flask Container
│   └── MySQL Container
```

---

# 10. Workflow Diagram

```text
Developer
   ↓
GitHub Repository
   ↓
Jenkins Pipeline
   ↓
Docker Build
   ↓
Docker Compose
   ↓
Flask + MySQL Containers
   ↓
AWS EC2 Deployment
```

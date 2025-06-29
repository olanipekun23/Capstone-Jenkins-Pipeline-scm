# CI/CD MASTERY Capstone Project

## Project Overview

A technology consulting firm is adapting a cloud architecture for its software application. As a DevOps Engineer, your task is to design and implement a robust CI/CD pipeline using Jenkins to automate the deployment of a web application. The goal is to achieve continuous integration, continuous deployment, and ensure scalability and reliability of the application.

---

## Pre-requisites

* Completion of Introduction to Jenkins
* Jenkins Freestyle Project
* Jenkins Pipeline Job Mini Project
* Understanding of Docker and GitHub

---

## Project Deliverables

### 1. Documentation:

* Jenkins setup instructions
* Plugin configuration
* Security configurations

### 2. Demonstration:

* Live CI/CD pipeline execution

---

## PROJECT COMPONENT 1: Jenkins Server Setup

### Objective:

Configure Jenkins server for CI/CD pipeline automation.

### Steps:

1. Install Jenkins on your Ubuntu WSL:

   ```bash
   sudo apt update
   sudo apt install openjdk-17-jdk
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins
   ```
2. Start Jenkins:

   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```
3. Access Jenkins at `http://localhost:8080` or via `ngrok` (e.g. `ngrok http 8080`)
4. Retrieve the initial admin password:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
5. Install the suggested plugins and set up your admin user.

### Security Measures:

* Enable Jenkins security under `Manage Jenkins > Configure Global Security`
* Disable CLI over Remoting
* Install and configure Role-Based Access Control (RBAC) plugin

---

## PROJECT COMPONENT 2: Source Code Management (SCM) Integration

### Objective:

Connect Jenkins to GitHub for source code management.

### Steps:

1. Install **Git** and **GitHub Integration Plugin** in Jenkins.
2. Add your GitHub repository credentials:

   * `Manage Jenkins > Credentials > Global > Add Credentials`
   * Use Personal Access Token (PAT) from GitHub
3. Configure webhook in your GitHub repo:

   * Go to **Settings > Webhooks**
   * Payload URL: `http://<your_ngrok_URL>/github-webhook/`
   * Content type: `application/json`
   * Enable: `Push events`

### Jenkins Configuration:

* In your Jenkins job, use the Git URL of your repository.
* Select `GitHub hook trigger for GITScm polling`

---

## PROJECT COMPONENT 3: Jenkins Freestyle Job for Build and Unit Tests

### Objective:

Create a Jenkins Freestyle job to build a web app and run unit tests.

### Steps:

1. Create a Freestyle Job: `New Item > Freestyle Project`
2. Under Source Code Management:

   * Select Git
   * Enter your GitHub repo URL and credentials
3. Build Trigger:

   * Enable GitHub hook trigger
4. Build Step:

   * Add `Execute Shell`
   * Example build command:

     ```bash
     echo "Running build and test..."
     docker build -t mywebapp .
     docker run --rm mywebapp
     ```
5. Save and build

---

## PROJECT COMPONENT 4: Jenkins Pipeline for Web Application

### Objective:

Develop a scripted pipeline for deploying a web app.

### Jenkinsfile Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t mywebapp .'
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d -p 8081:80 mywebapp'
                }
            }
        }
    }
}
```

---

## PROJECT COMPONENT 5: Docker Image Creation and Registry Push

### Objective:

Automate creation of Docker images and push to Docker Hub.

### Jenkins Pipeline Additions:

```groovy
stage('Login to DockerHub') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
    }
}

stage('Push to DockerHub') {
    steps {
        script {
            sh 'docker tag mywebapp $DOCKER_USER/mywebapp:latest'
            sh 'docker push $DOCKER_USER/mywebapp:latest'
        }
    }
}
```

---

## Final Notes

* Ensure Docker is accessible to Jenkins: `sudo usermod -aG docker jenkins`
* Restart Jenkins or shell session if permissions were updated
* Always secure sensitive credentials using Jenkins Credentials Manager

---

## ✅ Live Demo

* Use `ngrok` to expose Jenkins on the internet

  ```bash
  ngrok http 8080
  ```
* Use webhook trigger to start the pipeline automatically on GitHub push
* Access deployed web app at `http://localhost:8081` or via `ngrok http 8081`

---

## 🎉 Congratulations

You’ve completed the CI/CD MASTERY Capstone Project using Jenkins and Docker!

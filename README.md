CI/CD Demonstration with Jenkins, SonarQube, and Docker
This repository contains a demonstration of a CI/CD pipeline using Jenkins, SonarQube, Docker, and an HTML-based to-do list application. The CI/CD pipeline is set up on AWS EC2 instances.

Table of Contents
Introduction
Architecture
Setup Instructions
Prerequisites
EC2 Instances Setup
Jenkins Setup
SonarQube Setup
Docker Setup
Pipeline Workflow
Conclusion
Introduction
This project demonstrates a CI/CD pipeline that automates the process of building, testing, and deploying a simple to-do list web application. The pipeline utilizes:

Jenkins: for continuous integration and continuous deployment.
SonarQube: for code quality analysis.
Docker: for containerizing the application and deployment.
Architecture
The architecture involves the following EC2 instances:

Jenkins Server: Manages the CI/CD pipeline.
SonarQube Server: Performs static code analysis.
Docker Host: Runs the Docker containers for the application.
Setup Instructions
Prerequisites
AWS account with permissions to create EC2 instances.
Basic knowledge of Jenkins, SonarQube, Docker, and SSH.
EC2 Instances Setup
Create EC2 Instances

Launch three EC2 instances: one for Jenkins, one for SonarQube, and one for Docker.
Choose an appropriate instance type (e.g., t2.micro for testing).
Configure security groups to allow necessary ports:
Jenkins: TCP 8080
SonarQube: TCP 9000
Docker: TCP 2376 (if needed for remote management)
Install Required Software

Jenkins Instance:
sh
Copy code
sudo apt update
sudo apt install openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
SonarQube Instance:
sh
Copy code
sudo apt update
sudo apt install openjdk-11-jdk
sudo adduser sonar
sudo su - sonar
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.0.43852.zip
unzip sonarqube-8.9.0.43852.zip
exit
sudo mv sonarqube-8.9.0.43852 /opt/sonarqube
sudo chown -R sonar:sonar /opt/sonarqube
sudo nano /etc/systemd/system/sonarqube.service
Add the following to the sonarqube.service file:
makefile
Copy code
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

[Install]
WantedBy=multi-user.target
sh
Copy code
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
Docker Instance:
sh
Copy code
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce
sudo systemctl start docker
sudo systemctl enable docker
Jenkins Setup
Access Jenkins

Open a web browser and navigate to http://<Jenkins-Instance-IP>:8080.
Complete the setup wizard and install suggested plugins.
Create an admin user.
Install Required Plugins

Navigate to Manage Jenkins -> Manage Plugins and install:
Git
Docker Pipeline
SonarQube Scanner
Configure SonarQube in Jenkins

Navigate to Manage Jenkins -> Configure System.
Add a new SonarQube server with the URL http://<SonarQube-Instance-IP>:9000.
Create Jenkins Pipeline

Create a new pipeline job in Jenkins.
Define the pipeline script (Jenkinsfile):
groovy
Copy code
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-credentials'
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo/todo-app.git'
            }
        }
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('todo-app:latest')
                }
            }
        }
        stage('Deploy to Docker') {
            steps {
                sh 'docker run -d -p 80:80 todo-app:latest'
            }
        }
    }
}
SonarQube Setup
Access SonarQube

Open a web browser and navigate to http://<SonarQube-Instance-IP>:9000.
Complete the setup wizard and create an admin user.
Generate a token for Jenkins to use for authentication.
Create a Project

Create a new project in SonarQube.
Note the project key and use it in the Jenkins pipeline script.
Docker Setup
Prepare Dockerfile
Ensure your to-do list application repository contains a Dockerfile:
dockerfile
Copy code
FROM nginx:alpine
COPY . /usr/share/nginx/html
Pipeline Workflow
Commit Code Change

Push a change to the repository, such as modifying a .txt file.
Automated Pipeline Execution

Jenkins detects the change and triggers the pipeline.
Code is cloned, analyzed by SonarQube, built into a Docker image, and deployed.
Access Deployed Application

Open a web browser and navigate to http://<Docker-Instance-IP> to see the deployed to-do list application.
Conclusion
This demonstration showcases how to set up a CI/CD pipeline using Jenkins, SonarQube, and Docker on AWS EC2 instances. This setup helps in automating the software development lifecycle, ensuring code quality, and facilitating continuous delivery.

Make sure to replace placeholders (like <Jenkins-Instance-IP>, <SonarQube-Instance-IP>, etc.) with actual IP addresses or hostnames. Adjust the pipeline script as needed based on your specific setup and requirements.

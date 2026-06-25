Project-CICD 🚀

A complete end-to-end CI/CD pipeline for a Node.js static web application, automated using Jenkins, Docker, SonarQube, and AWS ECR.


📌 Table of Contents


Overview
Architecture
Tech Stack
Project Structure
Application
Dockerfile
Pipeline Stages
Prerequisites
Jenkins Credentials Required
Setup & Configuration
How to Run Locally



Overview

This project automates the full software delivery lifecycle of a Node.js web app:


Serves static files (HTML/CSS/JS) via Express.js on port 3000
Containerized using Docker with a lightweight node:18-alpine base image
Code quality enforced via SonarQube analysis and Quality Gate
Docker image pushed to AWS ECR with build-number tagging
Final container deployed locally on the Jenkins agent



Architecture

Developer pushes code to GitHub
            │
            ▼
    Jenkins Pipeline
            │
            ├── 1. Checkout Code         (GitHub → master branch)
            ├── 2. Build                 (Docker image: sample-app)
            ├── 3. SonarQube Analysis    (Static code analysis)
            ├── 4. Quality Gate          (Pass/Fail check)
            ├── 5. AWS Credentials Setup (Secure injection)
            ├── 6. Docker Push           (AWS ECR → tagged with BUILD_ID)
            └── 7. Docker Deployment     (Run container on port 3000)


Tech Stack

ToolVersionPurposeNode.js18+Application runtimeExpress.js4.18.2Static file serverDockerlatestContainerizationJenkinslatestCI/CD OrchestrationSonarQube9.9.xStatic Code AnalysisSonarScanner CLI4.8.1SonarQube ScannerAWS ECR-Docker Image RegistryAWS CLIlatestAWS Authentication


Project Structure

Project-cicd/
│
├── public/
│   └── index.html              # Static web page served by the app
│
├── apps.js                     # Express.js server entry point
├── package.json                # Node.js dependencies and scripts
├── dockerfile                  # Docker build instructions
├── Testing-app.jenkinsfile     # Jenkins declarative pipeline
└── README.md                   # Project documentation


Application

The app is a simple Node.js + Express static file server:


Serves all files from the public/ directory
Falls back to public/index.html for any unmatched routes
Listens on PORT env variable or defaults to 3000


Run locally:

bash# Install dependencies
npm install

# Start server
npm start

# Development mode (auto-restart on file change)
npm run dev

App will be available at: http://localhost:3000


Dockerfile

dockerfileFROM node:18-alpine       # Lightweight Node.js base image

WORKDIR /app              # Set working directory

COPY package.json .       # Copy dependency manifest
RUN npm install           # Install dependencies

COPY . .                  # Copy all source files

EXPOSE 3000               # Expose app port

CMD ["node", "apps.js"]   # Start the server

Build and run manually:

bashdocker build -t sample-app .
docker run -d -p 3000:3000 --name sample-app sample-app


Pipeline Stages

1. Checkout from Version Control

Clones the master branch from GitHub.

2. Build


Installs Docker on the Jenkins agent
Builds a Docker image tagged sample-app


3. SonarQube Analysis

Runs static analysis using SonarScanner CLI:


Project Name: My-website
Project Key: My-website


4. Quality Gate

Waits for SonarQube Quality Gate result (abortPipeline: false — pipeline continues regardless).

5. AWS Dir

Creates ~/.aws directory and credentials file if not present.

6. AWS Config


Injects AWS credentials securely from Jenkins credentials store
Writes credentials to ~/.aws/credentials
Verifies AWS access via aws s3 ls


7. Docker Push


Authenticates Docker with AWS ECR
Tags the image with BUILD_ID for versioning
Pushes to ECR repository nhytdfjynrsnydt in ap-south-1


573219416852.dkr.ecr.ap-south-1.amazonaws.com/nhytdfjynrsnydt:<BUILD_ID>

8. Docker Deployment


Stops and removes any existing sample-app container
Runs a fresh container on port 3000



Prerequisites

On Jenkins Server


Jenkins with these plugins installed:

Git Plugin
SonarQube Scanner Plugin
AWS Credentials Plugin
Docker Pipeline Plugin



Docker installed
AWS CLI installed
Java 11+
SonarScanner CLI 4.8.1 (must match SonarQube 9.9.x)


Install SonarScanner 4.8.1

bashcd /tmp
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.1.3023-linux.zip
sudo apt install unzip -y
sudo unzip sonar-scanner-cli-4.8.1.3023-linux.zip
sudo mv sonar-scanner-4.8.1.3023-linux \
  /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar-scanner
sudo chown -R jenkins:jenkins \
  /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar-scanner


Jenkins Credentials Required

Credential IDTypeDescriptionSonar-tokenSecret TextSonarQube authentication tokenaws-credsAWS CredentialsAWS Access Key ID & Secret Access Key


Setup & Configuration

SonarQube


Run SonarQube 9.9.x server
Create a project with key My-website
Generate an auth token → add to Jenkins as Sonar-token
Add SonarQube server in Jenkins → Manage Jenkins → Configure System → name it sonar-server
Add SonarScanner tool in Jenkins → Manage Jenkins → Global Tool Configuration → name it sonar-scanner


AWS ECR

Ensure the ECR repository exists:

bashaws ecr create-repository --repository-name nhytdfjynrsnydt --region ap-south-1


How to Run Locally

bash# Clone the repository
git clone https://github.com/Vipin-Patel-devops/Project-cicd.git
cd Project-cicd

# Install dependencies
npm install

# Start the app
npm start

# OR run with Docker
docker build -t sample-app .
docker run -d -p 3000:3000 --name sample-app sample-app

Visit: http://localhost:3000


Author

Vipin Patel — DevOps Engineer

GitHub: @Vipin-Patel-devops

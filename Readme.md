
# Chattingo - Real-time Chat Application

A modern, full-stack real-time chat application built with React, Spring Boot, and WebSocket technology. Features secure authentication, real-time messaging, and group chat functionality.

## 📋 Table of Contents

- [🚀 Project Overview](#-project-overview)
  - [Key Features](#key-features)
- [🛠️ Technology Stack](#️-technology-stack)
  - [Frontend](#frontend)
  - [Backend](#backend)
  - [Database](#database)
  - [DevOps & Infrastructure](#devops--infrastructure)
- [📦 Installation Instructions](#-installation-instructions)
  - [Setup VPS](#setup-vps)
    - [1. Setup Password-less SSH to VPS](#1-setup-password-less-ssh-to-vps)
    - [2. Install Docker & Docker Compose](#2-install-docker--docker-compose)
    - [3. Install Git](#3-install-git-if-not-present)
  - [Setup Jenkins](#setup-jenkins)
    - [1. Plugins](#1-plugins)
    - [2. Tools Installation](#2-tools-installation)
    - [3. SonarQube](#3-sonarqube)
    - [4. Credentials](#4-credentials)
    - [5. Global Trusted Libraries](#5-global-trusted-libraries)
    - [6. Declarative Pipeline](#6-declarative-pipeline)
    - [7. Github Webhook](#7-github-webhook)
- [🚀 Deployment Guide](#-deployment-guide)
  - [VPS Initial Setup](#vps-initial-setup)
- [🔧 Troubleshooting](#-troubleshooting)
  - [Common Issues and Solutions](#common-issues-and-solutions)
    - [Backend Issues](#backend-issues)
    - [Frontend Issues](#frontend-issues)
  - [Getting Help](#getting-help)
  - [Log Analysis](#log-analysis)
  - [Performance Optimization](#performance-optimization)
    - [Database Optimization](#database-optimization)
    - [Frontend Optimization](#frontend-optimization)
    - [Backend Optimization](#backend-optimization)

## 🚀 Project Overview

Chattingo is a comprehensive chat application designed for seamless real-time communication. The application provides a modern user interface with robust backend services, supporting individual and group messaging with real-time updates.

### Key Features

- **Real-time Messaging**: Instant message delivery using WebSocket technology
- **User Authentication**: Secure JWT-based authentication system
- **Group Chat**: Create and manage group conversations
- **User Management**: Profile management and user discovery
- **Responsive Design**: Mobile-friendly interface built with React and Tailwind CSS
- **Secure Communication**: End-to-end secure message transmission

## 🛠️ Technology Stack

### Frontend
- **React 18.2.0** - Modern JavaScript library for building user interfaces
- **Material-UI (MUI)** - React component library for consistent design
- **Tailwind CSS** - Utility-first CSS framework for styling
- **React Router DOM** - Client-side routing for single-page application
- **Redux Thunk** - State management for predictable application state
- **STOMP.js & SockJS** - WebSocket client libraries for real-time communication
- **React Icons** - Comprehensive icon library

### Backend
- **Spring Boot 3.3.1** - Java framework for building production-ready applications
- **Spring Security** - Authentication and authorization framework
- **Spring Data JPA** - Data persistence layer with Hibernate
- **Spring WebSocket** - Real-time bidirectional communication
- **JWT (JSON Web Tokens)** - Secure token-based authentication
- **Maven** - Dependency management and build automation
- **Java 17** - Latest LTS version of Java

### Database
- **MySQL 8.0** - Relational database for data persistence
- **Hibernate** - Object-relational mapping (ORM) framework

### DevOps & Infrastructure
- **Docker & Docker Compose** - Containerization and orchestration
- **Jenkins** - Continuous Integration/Continuous Deployment (CI/CD)
- **Nginx** - Web server and reverse proxy
- **SonarQube** - Code quality analysis
- **Trivy** - Container security scanning
- **OWASP Dependency Check** - Security vulnerability scanning

## 📦 Installation Instructions
## Setup VPS
### 1. Setup Password-less SSH to VPS
On your Jenkins master, run:

```bash
ssh-keygen -t ed25519 -C "chattingo-vps" -f ~/.ssh/chattingo_vps
```

This gives you:

* Private key → `~/.ssh/chattingo_vps`
* Public key → `~/.ssh/chattingo_vps.pub`

Now copy the public key:
```bash
cat ~/.ssh/chattingo_vps.pub
```

Then Go to _Hostinger Dashboard > Settings > SSH Keys > Add SSH Key_ And paste the public key there.
I repeat the **public** key.

Now SSH to your VPS.
```bash
ssh -i ~/.ssh/chattingo_vps root@<your-vps-ip>
```

Then copy the private key (`~/.ssh/chattingo_vps`) and store it in Jenkins Credentials. Read [Credentials](#4-credentials) section below

---


### 2. Install Docker & Docker Compose

```bash
apt update && apt upgrade -y
apt install -y docker.io docker-compose
systemctl enable docker
systemctl start docker
```

Verify:

```bash
docker --version
docker-compose --version
```

---

### 3. Install Git (if not present)
On my VPS there was already git installed.

But if you don't have git installed, you can install it using the following command:
```bash
apt install -y git
```

---

## Setup Jenkins
Here are the steps to setup Jenkins:

### 1. Plugins
<b> Go to Jenkins Master and click on  <mark>Manage Jenkins --> Plugins --> Available plugins</mark> to install the below plugins: </b>
  - NodeJs Plugin
  - OWASP Dependency Check
  - SonarQube Scanner
  - Docker Plugin

After these plugins are installed, move to <mark>Manage Jenkins --> Tools </mark> (Jenkins Worker)
- **NodeJs**: Add a NodeJs installation with name `node:18` and version `18.Y.Z`

- **Maven**: Add a Maven installation with name `maven:3.9` and version `3.9.Z`

- **OWASP**: Add a OWASP installation with name `OWASP`. Check the `Install Automatically` box and choose any version from `github.com`

- **SonarQube**: Add a SonarQube Scanner installation with name `sonarqube` and `any` version you want

### 2. Tools Installation
- Docker
  ```bash
  sudo apt install docker.io -y
  sudo usermod -aG docker ubuntu && newgrp docker
  ```
- Trivy
  ```bash
  sudo apt-get install wget apt-transport-https gnupg lsb-release -y
  wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
  echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
  sudo apt-get update -y
  sudo apt-get install trivy -y
  ```

### 3. SonarQube
Here are the steps to setup SonarQube:
-  Run the SonarQube Container
    ```bash
    docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
    ```
- Login to SonarQube server and navigate to <mark>Administration --> Security --> Users --> Tokens</mark> and create the credentials for jenkins to integrate with SonarQube

- Add that token to Jenkins, read  [Credentials](#4-credentials) section below


### 4. Credentials
Go to <mark>Manage Jenkins --> Credentials --> Global</mark> and add the following credentials:
- **Dockerhub**: Generate a token with read/write access and add it to Jenkins with id `docker` and type `Username with password`

- **Github**: Generate a ssh key that can be used to clone and push to the repository. Add it to Jenkins with id `github-ssh-key` and type `SSH Username with private key`

- **VPS**: Read [this](#1-setup-password-less-ssh-to-vps
) section to setup pass-less SSH to your VPS. Add it to Jenkins with id `vps-ssh-key` and type `SSH Username with private key`



### 5. Global Trusted Libraries
<b> Go to Jenkins Master and click on  <mark>Manage Jenkins --> System</mark> and search for `Global trusted libraries`</b> and add the following library:
- Name: `Shared`
- Default Version: `main`
- Project Repository: `https://github.com/HasanAshab/Jenkins_SharedLib.git`

### 6. Declarative Pipeline
Add a Item of type `Folder` name it `chattingo`. Everything related to the project will be inside that folder.

Now create these pipelines:
- **orchestrator**: The pipeline that orchestrates service pipelines.
    - Repository URL: `https://github.com/HasanAshab/chattingo`
    - Script Path: `Jenkinsfile`
    - Check the `GitHub hook trigger for GITScm polling` box

- **frontend**: Pipeline responsible for CI/CD of frontend.
    - Repository URL: `https://github.com/HasanAshab/chattingo`
    - Script Path: `frontend/Jenkinsfile`

- **backend**: Pipeline responsible for CI/CD of backend.
    - Repository URL: `https://github.com/HasanAshab/chattingo`
    - Script Path: `backend/Jenkinsfile`

### 7. Github Webhook
Go to [chattingo](https://github.com/hasanashab/chattingo) repo and navigate to <mark>Settings --> Webhooks</mark> and add the following webhook:
- Payload URL: `https://<jenkins-server-ip>/github-webhook/`
- Content type: `application/x-www-form-urlencoded`


## 🚀 Deployment Guide
### VPS Initial Setup
First [Connect to your VPS](#1-setup-password-less-ssh-to-vps) do the following step by step:

- Clone [chatingo-compose](https://github.com/HasanAshab/chattingo-compose) repo (not the chattingo repo)
  ```bash
  git clone https://github.com/HasanAshab/chattingo-compose.git
  ```
- Go to the cloned directory
    ```bash
    cd chattingo-compose
    ```
- Run setup script with your mysql password (First give permission to the script).
  ```bash
  chmod +x setup.sh
  ./setup.sh <your db password>
  ```
- Finally run the docker compose
  ```bash
  docker-compose up -d
  ```


## 🔧 Troubleshooting

### Common Issues and Solutions

#### Backend Issues

**Issue: Communication link failure**

Check the SPRING_DATASOURCE_URL in backend/.env file. It should be in the format of `jdbc:mysql://<service_name>:<port>/<database_name>`

Note: <service_name> is the name of the MySQL service in Docker Compose.


**Issue: Application fails to start**
```bash
# Check Java version
java -version
# Should be Java 17 or higher

# Check if port 8080 is available
netstat -tulpn | grep :8080
# Kill process if port is occupied
sudo kill -9 $(lsof -t -i:8080)
```

**Issue: Database connection failed**
```bash
# Check MySQL service status
sudo systemctl status mysql

# Start MySQL if not running
sudo systemctl start mysql

# Verify database exists
mysql -u root -p -e "SHOW DATABASES;"

# Create database if missing
mysql -u root -p -e "CREATE DATABASE chattingo_db;"
```

**Issue: JWT token errors**
```bash
# Generate new JWT secret (64 characters)
openssl rand -base64 64

# Update JWT_SECRET in backend/.env
# Restart the application
```

#### Frontend Issues

**Issue: CORS errors in browser**
```bash
# Check CORS configuration in backend/.env
CORS_ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com

# Restart backend after changing CORS settings
cd backend && ./mvnw spring-boot:run
```

**Issue: API connection failed**
```bash
# Check if backend is running
curl http://localhost:8080/
```

**Issue: WebSocket connection failed**
```bash
# Check browser console for WebSocket errors
# Verify JWT token is valid
# Ensure backend WebSocket endpoint is accessible
curl -I http://localhost:8080/ws
```

### Getting Help

If you encounter issues not covered here:

1. **Check the logs** - Most issues are revealed in application logs
2. **Verify configuration** - Ensure all environment variables are set correctly
3. **Test components individually** - Isolate the problem to specific services
4. **Check network connectivity** - Verify services can communicate with each other
5. **Review documentation** - Refer to Spring Boot and React documentation


For additional support, please create an issue in the GitHub repository with:
- Detailed error description
- Steps to reproduce
- Environment information
- Relevant log outputs

#### Log Analysis
```bash
# View application logs
docker-compose logs -f backend

# View specific service logs
docker logs chattingo-backend-1 --tail 100

# Search for errors
docker-compose logs backend | grep ERROR
```

### Performance Optimization

#### Database Optimization
```sql
-- Add indexes for better query performance
CREATE INDEX idx_messages_chat_id ON messages(chat_id);
CREATE INDEX idx_messages_timestamp ON messages(timestamp);
CREATE INDEX idx_users_email ON users(email);
```

#### Frontend Optimization
```conf
# Add to nginx.conf for caching static filess
location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|ttf|woff|woff2)$ {
    expires 1y;
    access_log off;
    add_header Cache-Control "public";
}
```

#### Backend Optimization
```properties
# Add to application.properties for production
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
logging.level.org.hibernate.SQL=WARN
server.compression.enabled=true
```

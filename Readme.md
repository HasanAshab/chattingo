
# Installation instructions
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

- Add that token to Jenkins, read  [credentials](#4-credentials) section below


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


## Deployment Guide

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

# 🚀 Python CI/CD Pipeline with AWS & GitHub Actions 

![Docker](https://img.shields.io/badge/Docker-ready-blue)
![Python](https://img.shields.io/badge/Python-3.10+-green)
![SonarCloud](https://img.shields.io/badge/SonarCloud-passing-brightgreen)
![Snyk](https://img.shields.io/badge/Snyk-secure-orange)
![CI](https://img.shields.io/github/actions/workflow/status/Ayaan49/python-aws-cicd-pipeline/main.yml?branch=development)


A hands-on DevOps project demonstrating how to build, containerize, and deploy a Flask web app using GitHub Actions, Docker, and AWS (ECR + EC2) — with SonarCloud and Snyk integration.

---

## 🧩 Overview

This project implements a simple counter service that:

- **GET /** → shows current count  
- **POST /** → increments count  
- Persists state in `/data/counter.txt`  
- Builds & deploys automatically via **GitHub Actions**

---

## ⚙️ Tech Stack

| Component | Technology |
|------------|-------------|
| **Language** | Python + Flask + Gunicorn |
| **CI/CD** | GitHub Actions |
| **Cloud** | AWS ECR + EC2 |
| **Containerization** | Docker + Docker Compose |
| **Code Quality & Security** | SonarCloud + Snyk |

---

## 🧰 Prerequisites

- Accounts: GitHub, AWS, SonarCloud, Snyk  
- Tools: `git`, `docker`, `python3`, `aws-cli`  
- AWS IAM User: ECR + EC2 access

---

## 🧱 Local Setup

```bash
git clone https://github.com/Ayaan49/python-aws-cicd-pipeline.git
cd python-aws-cicd-pipeline
python3 -m venv venv
source venv/bin/activate
pip3 install flask
pip3 freeze > requirements.txt
python counter-app.py
```

### Test locally:

```bash
curl localhost:8080/
curl -X POST localhost:8080/
```

---

## 🐳 Docker Build & Run

```bash
docker build -t "counter-app-local:1.0.0" .
docker run -p 8080:8080 counter-app-local:1.0.0
```

---

## ☁️ AWS Setup

### 1. Create an ECR repository

```bash
aws ecr create-repository --repository-name python-aws-cicd-pipeline --region <your-region>
```

### 2. Launch an EC2 instance (Ubuntu 22.04)

Allow inbound ports **22 (SSH)** and **80 (HTTP)**.

**Install Docker & AWS CLI on EC2:**

```bash
# Install aws cli
sudo apt-get install -y awscli

# Update packages
sudo apt-get update -y

# Install dependencies
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# Add Docker’s official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker + Compose plugin
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# (Optional) Allow ubuntu user to run docker without sudo
sudo usermod -aG docker ubuntu

```

---

## 🤖 GitHub Actions (CI/CD)

- **Workflow file:** `.github/workflows/ci-cd.yml`  
- **Triggers:** pushes to `development` (change to `main` when ready)

### Pipeline steps:

1. SonarCloud scan (code quality)  
2. Determine next semantic version (tag)  
3. Build Docker image & push to ECR  
4. Snyk scan (image vulnerabilities)  
5. SSH to EC2 and deploy with Docker Compose

---

### 🔐 Required GitHub Secrets

| Secret | Description |
|--------|--------------|
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | IAM credentials |
| `AWS_REGION` | e.g., `us-east-1` |
| `SONAR_TOKEN` | SonarCloud token |
| `SNYK_TOKEN` | Snyk API token |
| `EC2_PEM_KEY` | EC2 private key contents (single line) |
| `EC2_HOST`, `EC2_USER` | EC2 Public IP & username (e.g., `ubuntu`) |
| `ACCESS_TOKEN_2` | GitHub Personal Access Token for releases |

---

## 🧩 docker-compose.yml

```yaml
version: '2.4'  # The last version of Docker Compose file format that directly supports mem_limit and cpus
services:
  counter-service:
    container_name: py-aws-cicd
    image: "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
    volumes:
      - ./data:/data
    ports:
      - "80:8080"
    restart: always
    mem_limit: 256M
    cpus: 0.6
```

### First run only (on EC2):

```bash
cd ~/docker
sudo chmod -R 777 ./data
docker compose up -d --force-recreate
```

---

## ✅ Verify Deployment

**Open in browser:**

```
http://<EC2-Public-IP>/
```

**Increment counter:**

```bash
curl -X POST http://<EC2-Public-IP>/
```

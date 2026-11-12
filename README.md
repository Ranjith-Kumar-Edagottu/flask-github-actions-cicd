# End-to-End Flask CI/CD Pipeline with GitHub Actions, Docker & AWS EC2

An automated CI/CD pipeline that builds, containerizes, and deploys a Python Flask web application to an AWS EC2 instance on every push to the main branch.

## Architecture & Pipeline Flow

```text
Developer (git push)
       │
       ▼
GitHub Repository (main branch)
       │
       ▼
Job 1: docker-build-push (GitHub-Hosted Ubuntu Runner)
  ├── Checkout source code
  ├── Authenticate with Docker Hub (via GitHub Secrets)
  └── Build & push Docker image to Docker Hub
       │
       ▼
Job 2: deploy (AWS EC2 Self-Hosted Runner)
  ├── Pull latest Docker image from Docker Hub
  ├── Stop & remove existing running container
  └── Start updated container on Port 80 (-p 80:80)
       │
       ▼
Live Web Application (http://<EC2_PUBLIC_IP>)

***Tech Stack***
 - Application: Python, Flask, Gunicorn (Production WSGI Server)

 - Containerization: Docker, Docker Hub

- CI/CD Automation: GitHub Actions (GitHub-hosted runner + EC2 self-hosted runner)

- Cloud Infrastructure: AWS EC2 (Ubuntu Linux)

***Repository Structure***

flask-github-actions-cicd/
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions CI/CD workflow definition
├── templates/
│   └── index.html            # Frontend landing page
├── app.py                    # Flask application entry point
├── Dockerfile                # Container build instructions
├── requirements.txt          # Python package dependencies
└── README.md                 # Project documentation

***How to Reproduce This Setup***

1. Configure GitHub Repository Secrets
   Navigate to Settings → Secrets and variables → Actions and add the following repository secrets:

   - DOCKERHUB_USERNAME — Your Docker Hub username

    - DOCKERHUB_TOKEN — Docker Hub Personal Access Token (Read & Write permissions)

2. Provision AWS EC2 & Self-Hosted Runner
    - Launch an Ubuntu EC2 instance and allow inbound HTTP (Port 80) and SSH (Port 22) in the Security Group.

    - Install Docker on the EC2 instance and grant permissions to the ubuntu user:

sudo apt update && sudo apt install -y docker.io
sudo usermod -aG docker ubuntu

   - Register the EC2 instance as a self-hosted runner under Settings → Actions → Runners and run it as a background systemd      service:

sudo ./svc.sh install
sudo ./svc.sh start

3. Trigger Automated Deployment
   Push any commit to the main branch:

git add .
git commit -m "Deploy updated application"
git push origin main

--GitHub Actions automatically builds the new image, pushes it to Docker Hub, and deploys the updated container on EC2.

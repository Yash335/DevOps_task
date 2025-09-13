# 🚀 Deployment Guide: GitHub → Jenkins → EC2 → Docker

## 📌 Overview
This guide explains how code changes in a GitHub repository are automatically deployed to an **EC2 instance** using **Jenkins CI/CD** and **Docker**.

---

## 🔑 Prerequisites
- A GitHub repository containing:
  - Application source code
  - `Dockerfile`
- Jenkins server running on **localhost:8080**, with:
  - Git plugin
  - SSH Agent plugin
  - Credentials configured for:
    - **GitHub**
    - **EC2 SSH key**
    - **DockerHub** (`username + password`)
- Cloudflare Tunnel configured to expose Jenkins on a public URL (required because localhost cannot be used in GitHub webhook)
- EC2 instance with:
  - Docker installed
  - Proper SSH access from Jenkins
  - Security group allowing inbound traffic on the app port (3000)

---

## ⚙️ Deployment Flow

### 1. Push Code to GitHub
- Developer commits and pushes changes to the `main` branch:


 1. git add .
 2. git commit -m "New feature update"
 3. git push origin main


### 2. Jenkins Job Triggered via Webhook
- GitHub webhook triggers Jenkins on your **Cloudflare public URL**.
- Jenkins job performs the following actions:
  1. Checks out the latest code from GitHub using the configured credentials.
  2. Establishes SSH connection to the EC2 instance using the configured SSH key.
  3. Build, Push & Deploy Docker Image on EC2
  4. Application can be accessed on htps://<EC2_PUBLIC_IP>:3000


## ✅ Verification

- Run `docker ps` on EC2 → Check if container `devops-task` is running.
- Run `docker logs devops-task` → Verify app logs.
- Open the app in a browser at `http://<EC2_PUBLIC_IP>:3000`.

---

## 🔧 Troubleshooting

- **Port not accessible?**  
  Check EC2 security group → Ensure inbound rule allows port 3000.

- **Docker push denied?**  
  Verify DockerHub credentials in Jenkins.

- **Container not starting?**  
  Run `docker logs devops-task` to debug issues.

- **Webhook not triggering Jenkins?**  
  Ensure the Cloudflare Tunnel URL is reachable and correctly configured in GitHub webhook.


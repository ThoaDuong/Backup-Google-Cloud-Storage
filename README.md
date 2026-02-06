# Google Cloud Server Setup & Migration Guide

## Overview
This document describes how to:
- Create a Google Cloud VM
- Install Docker, Node.js
- Set up SSH & GitHub access
- Backup & Restore Google Cloud Storage
- Install SSL with Certbot

---

## 1️⃣ Create VM Instance (Google Cloud)

### VM Configuration
- Region: asia-southeast1 (Singapore)
- Machine type: E2 – 2 vCPU, 1 core, 4GB RAM
- OS: Ubuntu 22.04 LTS
- Disk size: 30 GB
- Networking:
  - Allow HTTP traffic
  - Allow HTTPS traffic
- Security:
  - Add SSH key
  - Set up SSH from your terminal
  ```bash
  gcloud compute ssh VM_SERVER_NAME@VM_NAME --zone VM_ZONE 
  ```
  - Open SSH key file
  ```bash
  cat ~/.ssh/google_compute_engine.pub
  ```
  - Paste the serverSSH key to the server

### Connect to Server
```bash
ssh -i ~/.ssh/google_compute_engine LOCAL-USERNAME@0.0.0.0
```

---

## 2️⃣ Update Ubuntu System
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release software-properties-common
```

---

## 3️⃣ Install Git
```bash
sudo apt install -y git
git --version
```

---

## 4️⃣ Install Docker Engine

### Create keyrings directory
```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

### Add Docker GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Add Docker repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 5️⃣ Add User to Docker Group
```bash
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker $USER
newgrp docker
```

Check:
```bash
docker --version
docker compose version
```

---

## 6️⃣ Install Node.js (Node 20 LTS)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

---

## 7️⃣ SSH Key for GitHub

### Generate SSH key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Start ssh-agent & add key
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

### Test GitHub
```bash
ssh -T git@github.com
```

---

## 8️⃣ Google Cloud Service Account Key
- IAM & Admin → Service Accounts
- Create service account (Role: Owner)
- Create key → google_cloud_key.json
- Add key to backend environment

---

## 9️⃣ Backup & Restore Google Cloud Storage

### Backup
```bash
gcloud auth login
mkdir -p ~/Backup_gcs/storage_old
gcloud storage rsync -m -r gs://storage ~/Backup_gcs/storage_old
```

### Restore
```bash
gcloud storage rsync -m -r ~/Backup_gcs/storage_old gs://storage
```

---

## 1️⃣0️⃣ SSL with Certbot
```bash
sudo apt install -y certbot

sudo certbot certonly \
  --standalone \
  -d domain.com \
  -d app.domain.com \
  --agree-tos \
  -m mail@gmail.com
```

Copy SSL:
```bash
mkdir -p ssl
sudo cp /etc/letsencrypt/live/domain.com/fullchain.pem ssl/
sudo cp /etc/letsencrypt/live/domain.com/privkey.pem ssl/
```

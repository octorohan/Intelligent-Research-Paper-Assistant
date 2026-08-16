# Docker Deployment Guide

Replace all `<placeholders>` with your actual values before running.

---

## Step 1 — Build the image

```bash
docker build -t <your-image-name> .
```

Example:
```bash
docker build -t rag-paper-app .
```

---

## Step 2 — Log in to Docker Hub

```bash
docker login
```

Enter your Docker Hub username and password when prompted.

---

## Step 3 — Tag the image for Docker Hub

```bash
docker tag <your-image-name> <your-dockerhub-username>/<your-image-name>:<tag>
```

Example:
```bash
docker tag rag-paper-app john/rag-paper-app:latest
```

---

## Step 4 — Push to Docker Hub

```bash
docker push <your-dockerhub-username>/<your-image-name>:<tag>
```

Example:
```bash
docker push john/rag-paper-app:latest
```

---

## Step 5 — On EC2: Pull and run the container

SSH into your EC2 instance, then:

# One-time setup: create the files/folders on the HOST before first run.
# If these don't exist, Docker will create them as directories instead of
# files when bind-mounting, which breaks sessions.json and checkpoints.db.
mkdir -p ~/rag-paper-app-data/embedding_cache
touch ~/rag-paper-app-data/sessions.json
touch ~/rag-paper-app-data/checkpoints.db

```bash
# Pull the image
docker pull <your-dockerhub-username>/<your-image-name>:<tag>

# Run the container
docker run -d \
  -p 8501:8501 \
  -e GOOGLE_API_KEY=<your-google-api-key> \
  -e TAVILY_API_KEY=<your-tavily-api-key> \
  -e QDRANT_URL=<your-qdrant-url> \
  -e QDRANT_API_KEY=<your-qdrant-api-key> \
  -e APP_PASSWORD=<your-chosen-password> \
  -v ~/rag-paper-app-data/embedding_cache:/app/embedding_cache \
  -v ~/rag-paper-app-data/sessions.json:/app/sessions.json \
  -v ~/rag-paper-app-data/checkpoints.db:/app/checkpoints.db \
  --restart unless-stopped \
  --name irpa-app \
  <your-dockerhub-username>/<your-image-name>:<tag>
```

The `-d` flag runs the container in the background.

---

## Accessing the app

Open your browser and go to:

```
http://<your-ec2-public-ip>:8501
```

> Make sure port **8501** is open in your EC2 security group inbound rules.

---

## Useful commands

```bash
# View running containers
docker ps

# View container logs
docker logs rag-paper-app

# Stop the container
docker stop rag-paper-app

# Remove the container
docker rm rag-paper-app
```

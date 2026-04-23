# Cloud Battle Arena 🌐

A real-time multiplayer Agar.io-style game deployed on Google Cloud Platform.

## Architecture
Firebase Hosting (Frontend) → GCP Compute Engine VM (Node.js WebSocket Server)

## Tech Stack
- **Backend:** Node.js WebSocket game server (OgarII)
- **Frontend:** HTML/CSS/JS browser client
- **Containerization:** Docker
- **Cloud:** GCP Compute Engine, Artifact Registry, VPC Firewall, Firebase Hosting

## Deployment Steps
1. Build and push Docker image to GCP Artifact Registry
2. Deploy container on Compute Engine VM
3. Configure VPC firewall to allow TCP on port 8080
4. Host frontend on Firebase Hosting
5. Connect browser frontend to VM external IP

## Local Setup
```bash
git clone https://github.com/umaarlll/cloud-battle-arena
cd cloud-battle-arena/OgarII
docker build -t agar-server-local .
docker run -d -it -p 8080:8080 --name my-local-server agar-server-local
```
Open `Cigar/www/index.html` in Chrome and enter `127.0.0.1:8080` as the server IP.

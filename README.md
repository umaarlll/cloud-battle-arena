## Demo

### Local Docker Run
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f811b184-6ba8-429c-b5e1-eb5587eff4b1" />

### Game Running in Browser
<img width="1600" height="852" alt="gdgoc-pic3" src="https://github.com/user-attachments/assets/713ed208-c5a2-47a7-9341-97f351d04b9b" />

# Cloud Battle Arena — Multiplayer Agar.io Clone on GCP

A fully deployed, cloud-hosted multiplayer browser game based on the Agar.io mechanic. Built and deployed end-to-end using Docker, Google Cloud Platform (Compute Engine + Artifact Registry), and Firebase Hosting — as part of a GDGoC UM Cloud Workshop.

---

## Architecture

The frontend is a static web client hosted on Firebase Hosting (HTTPS). It connects to the backend game server via WebSocket. The backend runs as a Docker container inside a Google Cloud Compute Engine VM (e2-micro, asia-southeast1 region), exposed on port 8080 through a custom VPC firewall rule. The Docker image is stored and versioned in GCP Artifact Registry, and the VM pulls it directly on boot.

---

## Tech Stack

The game server is built on OgarII, a Node.js WebSocket server. It is containerized with Docker and the image is stored in GCP Artifact Registry. The server runs on a GCP Compute Engine VM. The frontend is hosted on Firebase Hosting. Networking is handled through a GCP VPC firewall rule opening TCP port 8080, and permissions are managed via GCP IAM with the Artifact Registry Reader role attached to the VM service account.

---

## Deployment Overview

### 1. Local Testing

The game server was built and run locally using Docker to verify everything worked before pushing to the cloud.

```
cd OgarII
docker build -t agar-server-local .
docker run -d -it -p 8080:8080 --name my-local-server agar-server-local
```

The frontend was tested by opening Cigar/www/index.html in Chrome and connecting to 127.0.0.1:8080.

### 2. Build and Push to GCP Artifact Registry

After authenticating with GCP, the Docker image was built and pushed to a private Artifact Registry repository.

```
gcloud auth configure-docker asia-southeast1-docker.pkg.dev

docker build -t asia-southeast1-docker.pkg.dev/<PROJECT_ID>/agar-repo/agar-server:v1 .
docker push asia-southeast1-docker.pkg.dev/<PROJECT_ID>/agar-repo/agar-server:v1
```

### 3. Deploy on Compute Engine

A VM instance was created (e2-micro, asia-southeast1) and configured to deploy a container directly from Artifact Registry on boot. The Artifact Registry Reader IAM role was assigned to the VM's service account so it could pull the image. TCP port 8080 was opened via a custom VPC firewall ingress rule. The container was verified to be running via SSH and docker ps.

### 4. Deploy Frontend on Firebase

```
cd Cigar
firebase login
firebase init hosting
firebase deploy
```

The frontend is served over HTTPS. Players connect to the game server by entering the VM's external IP in the in-game menu.

Note: Because Firebase serves over HTTPS and the backend uses ws:// (unencrypted WebSocket), Chrome's insecure content policy must be manually allowed in site settings for the connection to work.

---

## How to Play

Open the Firebase URL, then allow insecure content in your browser's site settings to allow the HTTPS page to connect to the ws:// backend. Press Esc to open the server menu, enter YOUR_VM_EXTERNAL_IP:8080, set a nickname, and click Play.

---

## Admin / God Mode

SSH into the VM, attach to the running container, and use the OgarII console commands.

```
docker attach <container_id>

addbot 1 20               # Spawn 20 bots
setting pelletMinSize 50  # Giant food pellets
mass all 1000             # Max out all players
killall 1                 # Reset the map
```

To detach safely without killing the server, press Ctrl+P then Ctrl+Q.

---

## What I Learned

Working on this project gave me hands-on experience with the full containerization and cloud deployment workflow. I learned how to write a Dockerfile, build an image, and test it locally before pushing to the cloud. I got practical experience with GCP Artifact Registry for storing and versioning private Docker images, and with Compute Engine's container deployment feature which lets a VM pull and run a container directly on boot without manual setup. I also worked through IAM permissions — assigning the right service account role so the VM could access the registry — and configured VPC firewall rules to expose the correct port for external traffic. On the frontend side, I deployed a static client using the Firebase CLI and dealt with a real-world browser security constraint where Chrome blocks ws:// connections from HTTPS pages, requiring insecure content to be manually allowed.

---

## Cleanup

To avoid GCP charges after the workshop, delete the VM instance from Compute Engine and delete the repository from Artifact Registry.

---

## Project Structure

The repository has two main folders. OgarII contains the game server — a Node.js WebSocket server with a Dockerfile. Cigar/www contains the frontend, which is a static index.html and supporting assets.

---

## Credits

Based on the OgarII open-source Agar.io server implementation by Luka967.
Workshop organized by GDGoC UM — Cloud Workshop Series.

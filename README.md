# ☁️ GDGoC UM Cloud Workshop: Build Your Own Agar.io Clone (UM Arena)

Welcome to the UM Arena Cloud Workshop! In this hands-on session, you will deploy a full-stack multiplayer game. You will learn the complete development lifecycle: testing locally, using Docker and Google Cloud Platform (GCP) to host your backend game server, and using Firebase to host your web client.

By the end of this guide, you will have your very own game arena and the **"God Mode"** powers to control it.

---

## 🛠️ Prerequisites

Before we start, ensure you have the following installed:

* Google Cloud Account (with credits)
  * Google Cloud sign-up/free tier: https://cloud.google.com/free
* Google Cloud CLI (`gcloud`)
  * https://docs.cloud.google.com/sdk/docs/install-sdk#windows
* Code Editor (Recommended) : Visual Studio Code
  * Install VSCode: https://code.visualstudio.com/
  * Install Docker Extension in VSCode or any IDE: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker
* Docker Desktop (must be running)
  * https://www.docker.com/products/docker-desktop/
* Node.js
  * https://nodejs.org/en/download
* Git
  * https://git-scm.com/install/windows
* Firebase CLI

Install Firebase CLI in terminal:

```bash
npm install -g firebase-tools
```

---

## 💻 Part 1: Local Testing (The Dry Run)

### 1. Clone the Repository

Open a terminal in your preferred folder (For windows explorer, right click empty space and choose "Open In Terminal")

Clone the repository by typing the following command:
```bash
git clone https://github.com/JunBin05/cloud-battle-arena
cd cloud-battle-arena
```

### 2. Build and Run Backend Locally

```bash
cd OgarII
docker build -t agar-server-local .
docker run -d -it -p 8080:8080 --name my-local-server agar-server-local
```

### 3. Test Frontend Connection

* Go to `Cigar/www`
* Open `index.html` in Chrome
* Press "Esc" to bring up the menu
* Enter server IP: `127.0.0.1:8080`
* Click **Play**

If you spawn successfully → you're ready for the cloud 🚀

---

## 🐳 Part 2: Build & Push Backend to GCP

### 1. Create Artifact Registry

* Go to **Artifact Registry** in GCP
* Create repository:

  * Name: `agar-repo`
  * Format: Docker
  * Region: e.g. `asia-southeast1`

### 2. Build & Push Image

```bash
gcloud auth configure-docker asia-southeast1-docker.pkg.dev

docker build -t asia-southeast1-docker.pkg.dev/<PROJECT_ID>/agar-repo/agar-server:v1 .

docker push asia-southeast1-docker.pkg.dev/<PROJECT_ID>/agar-repo/agar-server:v1
```

If an docker image show up in your artiface registry repository, you are good to go!!

---

## 🚀 Part 3: Deploy Backend (Compute Engine)

### 1. Add Artifact Registry Reader role to Service Account

* Go to **IAM & Admin → IAM → Grant Access**
* Grant access to your project
  * New Principals: `[your_project_number]-compute@developer.gservice.com`. Your project number can be viewed at top right corner `⋮` icon > Project settings.
  * Role: `Artiface Registry Reader`
  * Click **Save**
* A new principal should appear at the list below.

### 2. Create VM & Deploy Container

* Go to **Compute Engine → VM Instances → Create Instance**
* Name: `agar-arena-vm`
* Machine type: `e2-micro` or `e2-medium`

#### Container Settings

* Click **Deploy Container**
* Image:

```
asia-southeast1-docker.pkg.dev/<PROJECT_ID>/agar-repo/agar-server:v1
```

Enable:

* Allocate a pseudo-TTY (-t)
* Keep STDIN open (-i)

#### Firewall

* Allow HTTP
* Allow HTTPS

Click **Create**

---

### 3. Open Port 8080

* Click `⋮` icon of your VM instance created, then click **View Network Details**
* Click on **Create VPC Firewall rules** Under VPC firewall rules section

* Create rule:

  * Name: `allow-agar-8080`
  * Direction: `Ingress`
  * Action: `Allow`
  * Target: `All instances in the network`
  * Source IPv4 range: `0.0.0.0/0`
  * Protocol: TCP `8080`

You may ssh into the VM to check if the container start up successfully (may take 1-2 minutes). If a red box error shows up, make sure you have done **Step 1**, then stop and start the vm again.



If booted up completed successfully, yellow box shows up when you ssh into vm. The commmand below should show that **[path_to_registry]/agar-server** is running.

```bash
docker ps
```



---

## 🌐 Part 4: Deploy Frontend (Firebase)

### 1. Initialize Firebase

```bash
firebase login
firebase init hosting
```

Use these settings:

* Public directory: `www`
* Single-page app: `No`
* GitHub deploys: `No`
* Overwrite index.html: **No (IMPORTANT)**

### 2. Deploy

```bash
firebase deploy
```

You’ll get a URL like:

```
https://your-project.web.app
```

---

## 🎮 Part 5: Play the Game

### ⚠️ Enable Insecure Content (Important)

Because Firebase uses HTTPS and your backend uses `ws://`, Chrome blocks it initially.

Steps:

1. Open your Firebase URL
2. Click 🔒 icon
3. Go to **Site Settings**
4. Allow **Insecure Content**
5. Refresh page

### Connect to Server

* Press **Esc** to bring up the menu
* Enter your VM External IP:

```
<YOUR_VM_IP>:8080
```

* Enter nickname → Click **Play**

---

## ⚡ Bonus: God Mode (Admin Console)

### 1. SSH into VM

Go to **Compute Engine → VM Instances → SSH**

### 2. Find Container

Displays currently running container.
If you tick the **Run as priviledged** box, you might have to add "sudo " infront of the command.

```bash
docker ps
```

Take note of currently running **container_id** OR **name**

### 3. Attach to Game Server

```bash
docker attach <CONTAINER_ID_OR_NAME>
```

Press Enter if blank.

---

### 🎛️ Admin Commands

```bash
addbot 1 20        # Spawn 20 bots
setting pelletMinSize 50   # Giant food
mass all 1000      # Everyone becomes huge
killall 1          # Reset map
```

---

### ⚠️ Exit Safely

**DO NOT press Ctrl + C** (kills server)

Instead:

```
Ctrl + P → Ctrl + Q
```

If you accidentally killed the server, do this:

```bash
docker image ls
docker run -d -it -p 8080:8080 --name agar-server <Insert_container-id_or_name>
```

---

## 🎉 You're Done!

You now have your own cloud-hosted Agar.io-style multiplayer game. Invite friends, tweak physics, and rule your arena like a digital overlord 👑

Happy hacking!

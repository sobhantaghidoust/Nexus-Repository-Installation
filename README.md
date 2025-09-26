README.md

# Nexus Repository Manager — Installation (Native & Docker)

This repository provides step-by-step instructions for installing **Sonatype Nexus Repository Manager 3** in two ways:

Native && Docker
---

## Prerequisites

- Native:
  - Linux (Ubuntu)
  - OpenJDK 17 
  - sudo privileges

- Docker:
  - Docker
  - Docker Compose 
---

## A) Native Installation (Linux + systemd)

### 1. Update packages and install Java
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

2. Create Nexus user

sudo useradd -r -s /bin/false nexus

3. Download and extract Nexus

cd /opt
sudo wget https://download.sonatype.com/nexus/3/nexus-3.83.2-01-linux-x86_64.tar.gz
sudo tar -xvzf nexus-3.83.2-01-linux-x86_64.tar.gz
sudo mv nexus-3.83.2-01 nexus

4. Set permissions

sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work

5. Create a systemd service

sudo nano /etc/systemd/system/nexus.service

Paste:

[Unit]
Description=Sonatype Nexus Repository Manager
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
Restart=on-abort

[Install]
WantedBy=multi-user.target

6. Enable and start service

sudo systemctl daemon-reload
sudo systemctl enable nexus
sudo systemctl start nexus
sudo systemctl status nexus

7. Get admin password

sudo cat /opt/sonatype-work/nexus3/admin.password

8. Access Web UI

Open:
http://<server-ip>:8081
Default user: admin

⸻

B) Docker Installation

1. Install Docker

sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

2. Run Nexus container

docker run -d -p 80:8081 --name nexus \
  -v nexus-data:/nexus-data \
  sonatype/nexus3

 • -p 80:8081: maps container port 8081 → host port 80
 • -v nexus-data:/nexus-data: persists Nexus data
 • --name nexus: container name

3. Configure auto-restart

docker update --restart=always nexus

4. Get admin password

docker exec -it nexus bash
cat /opt/sonatype-work/nexus3/admin.password

5. Access Web UI

Open:
http://<server-ip> (if mapped to port 80)
or
http://<server-ip>:8081

⸻

Notes
 • After logging in the first time, you will be prompted to change the admin password.
 • You can create hosted or proxy repositories from the Repositories menu in the Nexus Web UI.
 • Nexus supports uploading .deb, .rpm, Maven, npm, and many other package types.

⸻

License

MIT License © 2025 SobhanTaghidoust

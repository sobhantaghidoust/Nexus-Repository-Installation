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
> سُب:
Perfect — here’s a clean, exact step-by-step recipe you can copy-paste and follow. It assumes you want the Docker-Compose mode as the recommended (3rd) method. I include the exact docker/compose.yml, the commands to run, how to get the admin password, and a short troubleshooting section that covers the common problems (permission, wrong log mapping, port conflict). If anything in the logs looks wrong, follow the fixes shown — I already anticipated the errors you saw earlier and included the fixes.

⸻

Recommended (mode 3) — Docker Compose (step-by-step)

1) Install Docker (on Ubuntu)

sudo apt update
sudo apt upgrade -y
sudo apt install -y docker.io
# ensure docker is running
sudo systemctl enable docker
# recommended: install the compose plugin (if you don't have 'docker compose' command)
sudo apt install docker-compose

2) Create the compose file

Create folder docker/ in your repo (or on the server) and add docker-compose.yml with this content:

(vim/vi/nano)  docker-compose.yml
paste this:

version: "3.8"
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: always
    ports:
      - "80:8081"   # change to "8081:8081" if you prefer to use host port 8081
    volumes:
      - /home/nexus-data:/nexus-data   # host path for Nexus data
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

Tip: If you prefer not to use port 80 (maybe blocked by another service), change the ports line to "8081:8081" and open http://<server-ip>:8081.

3) Create the host data directory (quick safe step)

sudo mkdir -p /home/nexus-data
# Quick permissive fix so container can write the first time:
sudo chmod 0777 /home/nexus-data

(After you confirm Nexus works you can tighten permissions — instructions below.)

4) Start Nexus with Compose

From the folder that contains docker/compose.yml:

docker-compose up -d
---

## Troubleshooting

### Nexus container keeps restarting
If you see in docker ps that the Nexus container status is Restarting (or logs keep looping):

1. Check logs
   ```bash
   docker compose logs -f nexus

5) Get the initial admin password and open UI

# inside container:
docker exec -it nexus cat /nexus-data/admin.password

# then open in browser:
# if you used "80:8081" -> http://<server-ip>
# if you used "8081:8081" -> http://<server-ip>:8081
# login user: admin  password: (value from command above)


⸻

Quick stop/start/status commands

# stop
docker compose down

# start again
docker compose up -d

# check container list
docker ps

# follow logs again
docker compose logs -f nexus


⸻

Troubleshooting (common problems + fixes)

A) Error in logs: No such file or directory for /opt/sonatype/.../nexus.log or similar

Cause: you mapped a host folder to the wrong container path (for example /var/log/nexus) or created a conflicting mapping.
Fix:
 • Use only /nexus-data mapping (recommended). Remove any mapping to /var/log/nexus.
 • Update docker/compose.yml to the version above, then:

docker compose down
docker compose up -d

B) Permission denied / container cannot write to /nexus-data

Quick (safe for testing):

sudo chmod -R 0777 /home/nexus-data
docker compose restart nexus

Safer (production): find the UID used by the Nexus process inside the image and chown the directory to that UID, e.g.:

# get the container UID (runs a temporary container and prints user id)
docker run --rm --entrypoint id sonatype/nexus3
# then chown the host folder to that UID (replace <uid>:<gid> with values printed)
sudo chown -R <uid>:<gid> /home/nexus-data
sudo chmod -R 0755 /home/nexus-data
docker compose restart nexus

C) Port conflict (can’t bind to port 80)

Symptom: bind: address already in use or compose fails.
Fix:
 • Change the compose ports mapping to "8081:8081" and use http://<server-ip>:8081 in browser.
 • Or stop the service that uses 80 (e.g., sudo systemctl stop nginx).

D) docker compose command not found

If docker compose is missing, use the legacy command:

docker-compose up -d

Or install the plugin as in step 1.

E) Nexus starts but UI shows blank or login fails

> سُب:
• Check logs: docker compose logs -f nexus and look for exceptions.
 • If you see Started Sonatype Nexus but UI still not reachable, check firewall:

sudo ufw status
# allow port 80 or 8081
sudo ufw allow 80/tcp
sudo ufw allow 8081/tcp


⸻

Final notes (what I’d do if anything breaks)
 • If logs complain about missing log directory — I remove any incorrect log volume mapping and keep only /nexus-data.
 • If write errors — make sure /home/nexus-data exists and is writable (chmod 0777 quick fix), then find the correct UID and chown for a cleaner solution.
 • If port conflicts — switch to 8081 mapping.

⸻

If you want, I can:
 • produce the exact docker-compose.yml text ready to paste into GitHub (I already gave it above), or
 • give the single-line commands you can paste into the server to create the folder, create the file, and run compose (I can produce that exact shell snippet now).

⸻

Notes
 • After logging in the first time, you will be prompted to change the admin password.
 • You can create hosted or proxy repositories from the Repositories menu in the Nexus Web UI.
 • Nexus supports uploading .deb, .rpm, Maven, npm, and many other package types.

⸻

License

MIT License © 2025 SobhanTaghidoust

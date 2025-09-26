# Nexus Repository Manager — Installation (Native, Docker & Docker Compose)

This repository provides step-by-step instructions for installing Sonatype Nexus Repository Manager 3 in three ways:

1. Native installation (Linux + systemd)  
2. Docker (single container with `docker run`)  
3. Docker Compose (recommended)

Default Web UI: http://<server-ip>:8081 (or http://<server-ip> if mapped to port 80)  

Initial admin password is located in:
- Native: /opt/sonatype-work/nexus3/admin.password  
- Docker/Compose: /nexus-data/admin.password

## A) Native Installation (Linux + systemd)

### 1. Install prerequisites
```bash
sudo apt update
sudo apt install -y openjdk-17-jre curl tar wget
java -version

2. Create user and directories

sudo useradd --system --home /opt/nexus --shell /bin/false nexus
sudo mkdir -p /opt/nexus /opt/sonatype-work/nexus3
sudo chown -R nexus:nexus /opt/nexus /opt/sonatype-work

3. Download and extract Nexus

cd /opt
sudo wget https://download.sonatype.com/nexus/3/nexus-3.83.2-01-linux-x86_64.tar.gz
sudo tar -xvzf nexus-3.83.2-01-linux-x86_64.tar.gz
sudo mv nexus-3.83.2-01 nexus
sudo chown -R nexus:nexus /opt/nexus

4. Configure run user and JVM options

echo 'run_as_user="nexus"' | sudo tee /opt/nexus/bin/nexus.rc

echo 'INSTALL4J_ADD_VM_PARAMS="-Xms1200m -Xmx1200m -Dnexus-work=/opt/sonatype-work/nexus3 -Djava.io.tmpdir=/opt/sonatype-work/nexus3/tmp"' \
| sudo tee /opt/nexus/bin/nexus.vmoptions

sudo chown nexus:nexus /opt/nexus/bin/nexus.rc /opt/nexus/bin/nexus.vmoptions

5. Create systemd service

sudo tee /etc/systemd/system/nexus.service > /dev/null <<'EOF'
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
EOF

6. Enable and start service

sudo systemctl daemon-reload
sudo systemctl enable nexus
sudo systemctl start nexus
sudo systemctl status nexus

7. Get admin password

sudo cat /opt/sonatype-work/nexus3/admin.password


⸻

B) Docker Installation (single container)

1. Install Docker

sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl enable --now docker

2. Run Nexus container

docker run -d --name nexus \
  -p 80:8081 \
  -v nexus-data:/nexus-data \
  sonatype/nexus3:latest

3. Configure auto-restart

docker update --restart=always nexus

4. Get admin password

docker exec -it nexus cat /opt/sonatype-work/nexus3/admin.password

5. Logs

docker logs -f nexus


⸻

C) Docker Compose Installation (recommended)

1. Install Docker and Compose

sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable --now docker

2. Create docker/compose.yml

version: "3.8"
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: always
    ports:
      - "80:8081"   # host port 80 → container port 8081
    volumes:
      - /home/nexus-data:/nexus-data
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

3. Start Nexus

cd docker
docker compose up -d

4. Get admin password

docker exec -it nexus cat /nexus-data/admin.password

5. Access UI
 • If you mapped 80:8081: http://<server-ip>
 • If you mapped 8081:8081: http://<server-ip>:8081

⸻

Troubleshooting

Container keeps restarting

If you see Restarting in docker ps:
 1. Check logs:

docker compose logs -f nexus


 2. Fix permissions on host volume:

sudo mkdir -p /home/nexus-data
sudo chmod -R 0777 /home/nexus-data


 3. Recreate container:

docker compose down
docker compose up -d


 4. Check again:

docker compose logs -f nexus

Wait for:

Started Sonatype Nexus OSS ...



⸻

License

MIT License © 2025 SobhanTaghidoust

---

# Deployment Guide

## Step 1: Launch EC2 Instances

Create two EC2 instances:

```text
Jitsi EC2  -> web, prosody, jicofo, jvb
Jibri EC2  -> Jibri recorder
```

Recommended inbound rules for Jitsi EC2:

```text
22/tcp      from your IP only
80/tcp      from 0.0.0.0/0
443/tcp     from 0.0.0.0/0
10000/udp   from 0.0.0.0/0
5222/tcp    from Jibri EC2 public IP or Jibri security group only
ICMP         optional, from Jibri EC2 only if needed for testing
```

Jibri EC2 does not need public web ports. It mainly needs outbound access to the Jitsi server and the internet.

## Step 2: DNS

Create an A record:

```text
your-subdomain.your-domain.com -> Jitsi EC2 Elastic IP
```

Use the domain in `PUBLIC_URL`. Do not build the production setup around a raw public IP.

## Step 3: Install Docker

Install Docker Engine and Docker Compose plugin on both EC2 instances.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg unzip
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
docker --version
docker compose version

```

## Step 4: Start Jitsi

On the Jitsi EC2:

```bash
cd jitsi
cp .env.example .env
vi .env
docker compose -f docker-compose.yml up -d
```

Verify:

```bash
docker ps
```

Expected containers:

```text
web
prosody
jicofo
jvb
```

## Step 5: Start Jibri

On the Jibri EC2:

```bash
cd jibri
cp .env.example .env
vi .env
docker compose -f jibri.yml up -d
```

Verify:

```bash
docker ps
docker logs -f jibri
```

## Step 6: Connectivity Checks

From Jibri EC2:

```bash
ping your-domain.example.com
nc -zv your-domain.example.com 443
nc -zv your-domain.example.com 5222
```

If port 5222 fails, check the Jitsi EC2 security group and Prosody container.

## Step 7: Recording Test

Open Jitsi in the browser, create a room, become moderator, and start recording. Then check the Jibri recording directory:

```bash
sudo find ~/.jitsi-meet-cfg/jibri/recordings -type f
```

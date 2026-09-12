# Jitsi Meet + Jibri Deployment on AWS EC2 using Docker

This project documents a Docker-based deployment of Jitsi Meet on AWS EC2 with a separate Jibri recording server. The setup includes HTTPS, DNS, security group rules, Jibri recording configuration, and troubleshooting notes.

## Architecture
<img width="4182" height="1549" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/19560106-e68f-460a-978e-cb881049a8ce" />

## Main Features

- Jitsi Meet deployed with Docker Compose
- Separate Jibri EC2 instance for recording
- HTTPS configured with mounted SSL certificate files
- Jibri connected to Jitsi through XMPP port 5222
- Recording service enabled
- Notes for scaling Jibri containers

## Repository Structure

```text
.
├── docs/
│   ├── deployment-guide.md
├── jibri/
│   ├── .env.example
│   └── jibri.yml
├── jitsi/
│   ├── .env.example
│   └── docker-compose.yml
├── screenshots/
├── .gitignore/
├── LICENSE/
└── README.md/
```

## Important Security Notice

Real `.env` files, AWS PEM keys, SSL private keys, public IPs, instance IDs, and real passwords are not included. Use the `.env.example` files and generate your own secrets.

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/MutahirTayyab/jitsi-meet-production-setup.git
cd jitsi-meet-production-setup
```

### 2. Prepare Jitsi EC2

```bash
cd jitsi
cp .env.example .env
vi .env
```

Replace placeholders such as:

```env
PUBLIC_URL=https://your-domain.example.com
JVB_ADVERTISE_IPS=YOUR_JITSI_PUBLIC_IP
DOCKER_HOST_ADDRESS=YOUR_JITSI_PUBLIC_IP
Note: Generate your own Passwords! (./gen-passwords.sh)
```

Start Jitsi:

```bash
docker compose -f docker-compose.yml up -d
```

### 3. Prepare Jibri EC2

```bash
cd jibri
cp .env.example .env
vi .env
```

Make sure the Jibri passwords match the Jitsi `.env` values.

Start Jibri:

```bash
docker compose -f jibri.yml up -d
```

### 4. Verify

```bash
docker ps
nc -zv your-domain.example.com 443
nc -zv your-domain.example.com 5222
```

## Notes

1. One Jibri can handle one active recording. For four simultaneous recordings, four Jibri workers are required. On AWS EC2, running multiple Jibri containers on one host may have audio limitations because ALSA loopback may not be available on the AWS kernel.
2. It is recommended to scale recorders on any other server rather than any Cloud Server.
3. One more thing to note in our practical we have used AWS Instance type m7i-flex-large which is  not enough for multiple recorders. So use that instance type which is full of resources so that multiple recorders can easily run!

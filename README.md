# Yii2 + Docker Swarm + NGINX + Ansible + CI/CD

This repository contains a complete solution for deploying a Yii2 PHP application using Docker Swarm with NGINX as a reverse proxy, automated with Ansible and CI/CD via GitHub Actions.

## Architecture

- **Yii2 Application**: PHP application running in Docker containers
- **Docker Swarm**: Container orchestration for high availability
- **NGINX**: Host-based reverse proxy (not containerized)
- **Ansible**: Infrastructure automation
- **GitHub Actions**: CI/CD pipeline

## Prerequisites

- AWS EC2 instance (Ubuntu 22.04 recommended)
- GitHub account with repository access
- Docker Hub account (or GitHub Container Registry)
- Domain name pointed to your EC2 instance (optional)

## Setup Instructions

### 1. Infrastructure Setup

1. Launch an EC2 instance on AWS (t2.medium or larger recommended)
2. Configure security groups to allow:
   - SSH (port 22)
   - HTTP (port 80)
   - HTTPS (port 443)
   - Docker Swarm ports (2377, 7946, 4789)
3. Add your public key to the instance for SSH access

### 2. Repository Setup

1. Clone this repository
2. Update the following files with your specific details:
   - `ansible/inventory.ini`: Update with your EC2 instance IP and key file path
   - Docker Hub username in `docker-compose.yml`
   - Domain name in `nginx/yii2-app.conf`

### 3. GitHub Secrets Setup

Add the following secrets to your GitHub repository:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password
- `EC2_HOST`: Your EC2 instance IP address
- `EC2_USER`: SSH username (e.g., ubuntu)
- `SSH_PRIVATE_KEY`: Your SSH private key content

### 4. Ansible Deployment

Run the following command to set up the EC2 instance:

```bash
ansible-playbook -i ansible/inventory.ini ansible/setup.yml
```

This will:
- Install Docker, NGINX, and other requirements
- Initialize Docker Swarm
- Configure NGINX as a reverse proxy
- Deploy the Yii2 application

### 5. CI/CD Pipeline

The CI/CD pipeline will be triggered automatically on every push to the main branch. It will:
1. Build the Docker image
2. Push it to Docker Hub
3. Deploy the new image to your Docker Swarm
4. Roll back if the deployment fails

## Testing the Deployment

1. Access your application using your domain name or EC2 public IP
2. Verify that the Yii2 application loads correctly
3. Test a code change by modifying the application code, pushing to GitHub, and verifying that the changes are deployed automatically

## Monitoring (Bonus)

To add monitoring with Prometheus and Node Exporter:

1. Run:
```bash
ansible-playbook -i ansible/inventory.ini ansible/monitoring.yml
```

2. Access Prometheus at `http://your-server-ip:9090`

## Assumptions

- Ubuntu 22.04 LTS is used as the base OS
- SSH access to the EC2 instance is available
- The application does not require complex database migrations
- A subdomain or domain is available for the application

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy.yml
├── ansible/
│   ├── inventory.ini
│   ├── setup.yml
│   └── templates/
│       └── yii2-app.conf.j2
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── nginx/
│   └── yii2-app.conf
└── README.md
```

## Troubleshooting

- Check NGINX logs: `/var/log/nginx/error.log`
- Check Docker service logs: `docker service logs yii2app_app`
- Verify Docker Swarm status: `docker node ls` and `docker service ls`
- Ensure NGINX can communicate with Docker Swarm services

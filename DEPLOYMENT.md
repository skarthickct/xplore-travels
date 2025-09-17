# Deployment Guide

## Prerequisites

### AWS EC2 Setup
1. Launch Ubuntu EC2 instance
2. Install Docker:
```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
```

### GitHub Secrets Configuration

Navigate to repository Settings → Secrets and variables → Actions:

| Secret | Description | Example |
|--------|-------------|---------|
| `DOCKER_USERNAME` | Docker Hub username | `myusername` |
| `DOCKER_PASSWORD` | Docker Hub password/token | `dckr_pat_xxx` |
| `EC2_SSH_KEY` | Private key content | `-----BEGIN RSA PRIVATE KEY-----` |
| `EC2_HOST` | EC2 public IP | `54.123.45.67` |

## Deployment Process

### Automatic (GitHub Actions)

1. Push to `main` branch
2. Workflow triggers automatically
3. Builds optimized Docker image
4. Pushes to Docker Hub
5. Deploys to EC2 instance

### Manual Deployment

```bash
# 1. Build locally
cd app
docker build -t username/xplore-travels:latest .

# 2. Push to registry
docker push username/xplore-travels:latest

# 3. Deploy to EC2
ssh -i key.pem ubuntu@EC2_HOST
docker pull username/xplore-travels:latest
docker stop xplore-travels || true
docker rm xplore-travels || true
docker run -d --name xplore-travels -p 5000:5000 --restart unless-stopped username/xplore-travels:latest
```

## Environment Configuration

### Production Settings
- Port: 5000
- Workers: 2
- Timeout: 120s
- Restart policy: unless-stopped

### Security Considerations
- Non-root container execution
- Minimal base image (Alpine)
- No sensitive data in image
- SSH key rotation recommended

## Monitoring & Maintenance

### Health Checks
```bash
# Application status
curl http://EC2_HOST:5000/

# Container status
docker ps | grep xplore-travels

# Resource usage
docker stats xplore-travels
```

### Log Management
```bash
# View logs
docker logs xplore-travels

# Follow logs
docker logs -f xplore-travels

# Log rotation (if needed)
docker logs --tail 100 xplore-travels
```

### Updates
```bash
# Pull latest image
docker pull username/xplore-travels:latest

# Restart with new image
docker stop xplore-travels
docker rm xplore-travels
docker run -d --name xplore-travels -p 5000:5000 --restart unless-stopped username/xplore-travels:latest

# Clean old images
docker image prune -f
```

## Rollback Procedure

```bash
# List available images
docker images username/xplore-travels

# Rollback to previous version
docker stop xplore-travels
docker rm xplore-travels
docker run -d --name xplore-travels -p 5000:5000 --restart unless-stopped username/xplore-travels:previous-tag
```

## Scaling Options

### Horizontal Scaling
```bash
# Run multiple instances
docker run -d --name xplore-travels-1 -p 5001:5000 username/xplore-travels:latest
docker run -d --name xplore-travels-2 -p 5002:5000 username/xplore-travels:latest

# Use load balancer (nginx/ALB)
```

### Vertical Scaling
```bash
# Increase resources
docker run -d --name xplore-travels -p 5000:5000 --memory=1g --cpus=2 username/xplore-travels:latest
```

## Troubleshooting

### Common Issues

**Deployment fails**
- Check GitHub secrets
- Verify EC2 connectivity
- Ensure Docker is running

**Container won't start**
- Check port availability: `netstat -tulpn | grep 5000`
- Review container logs: `docker logs xplore-travels`
- Verify image integrity: `docker inspect username/xplore-travels:latest`

**Application errors**
- Database file permissions
- Static file access
- Environment variables

### Emergency Procedures

**Service down**
```bash
# Quick restart
docker restart xplore-travels

# Full redeploy
docker stop xplore-travels && docker rm xplore-travels
docker run -d --name xplore-travels -p 5000:5000 --restart unless-stopped username/xplore-travels:latest
```

**Disk space issues**
```bash
# Clean Docker resources
docker system prune -af
docker volume prune -f
```

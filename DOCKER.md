# Docker Configuration

## Multistage Dockerfile

### Stage 1: Builder
```dockerfile
FROM python:3.9-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN apk add --no-cache --virtual .build-deps gcc musl-dev && \
    pip install --no-cache-dir -r requirements.txt && \
    apk del .build-deps
```

### Stage 2: Runtime
```dockerfile
FROM python:3.9-alpine
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --chown=appuser:appgroup . .
USER appuser
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "--timeout", "120", "app:app"]
```

## Optimization Features

### Size Optimization
- **Alpine Linux**: Minimal base image (13MB vs 120MB)
- **Multistage build**: Removes build dependencies
- **Layer caching**: Optimized layer order
- **Final size**: 264MB (71% reduction from 922MB)

### Security Optimization
- **Non-root user**: Runs as `appuser` (UID 1001)
- **Minimal packages**: Only runtime dependencies
- **No shell access**: Limited attack surface
- **File permissions**: Proper ownership and permissions

### Performance Optimization
- **Gunicorn**: Production WSGI server
- **Multi-worker**: 2 worker processes
- **Timeout handling**: 120s request timeout
- **Auto-restart**: `unless-stopped` policy

## Build Commands

### Development Build
```bash
cd app
docker build -t xplore-travels:dev .
```

### Production Build
```bash
cd app
docker build -t xplore-travels:latest .
docker tag xplore-travels:latest username/xplore-travels:latest
```

### Build with Cache
```bash
docker build --cache-from xplore-travels:latest -t xplore-travels:new .
```

## Run Commands

### Basic Run
```bash
docker run -d -p 5000:5000 --name xplore-travels xplore-travels:latest
```

### Production Run
```bash
docker run -d \
  --name xplore-travels \
  -p 5000:5000 \
  --restart unless-stopped \
  --memory=512m \
  --cpus=1 \
  xplore-travels:latest
```

### Development Run
```bash
docker run -it \
  --name xplore-travels-dev \
  -p 5000:5000 \
  -v $(pwd):/app \
  xplore-travels:dev \
  python app.py
```

## Docker Compose

### docker-compose.yml
```yaml
version: '3.8'
services:
  xplore-travels:
    build: ./app
    ports:
      - "5000:5000"
    restart: unless-stopped
    environment:
      - FLASK_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Usage
```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## Image Management

### Tagging Strategy
```bash
# Version tags
docker tag xplore-travels:latest xplore-travels:v1.0.0
docker tag xplore-travels:latest xplore-travels:stable

# Environment tags
docker tag xplore-travels:latest xplore-travels:production
docker tag xplore-travels:latest xplore-travels:staging
```

### Registry Operations
```bash
# Push to Docker Hub
docker push username/xplore-travels:latest
docker push username/xplore-travels:v1.0.0

# Pull from registry
docker pull username/xplore-travels:latest

# List local images
docker images xplore-travels
```

## Debugging

### Container Inspection
```bash
# Container details
docker inspect xplore-travels

# Process list
docker exec xplore-travels ps aux

# File system
docker exec -it xplore-travels ls -la /app
```

### Log Analysis
```bash
# Application logs
docker logs xplore-travels

# Real-time logs
docker logs -f --tail 100 xplore-travels

# Log with timestamps
docker logs -t xplore-travels
```

### Interactive Debugging
```bash
# Shell access
docker exec -it xplore-travels sh

# Run commands
docker exec xplore-travels python -c "import app; print('OK')"

# Check processes
docker exec xplore-travels ps aux | grep gunicorn
```

## Performance Monitoring

### Resource Usage
```bash
# Real-time stats
docker stats xplore-travels

# Memory usage
docker exec xplore-travels cat /proc/meminfo

# CPU usage
docker exec xplore-travels top -n 1
```

### Health Checks
```bash
# HTTP health check
curl -f http://localhost:5000/ || echo "Service down"

# Container health
docker inspect --format='{{.State.Health.Status}}' xplore-travels

# Process health
docker exec xplore-travels pgrep gunicorn
```

## Cleanup

### Remove Containers
```bash
# Stop and remove
docker stop xplore-travels
docker rm xplore-travels

# Force remove
docker rm -f xplore-travels
```

### Remove Images
```bash
# Remove specific image
docker rmi xplore-travels:latest

# Remove all unused images
docker image prune -f

# Remove all xplore-travels images
docker rmi $(docker images xplore-travels -q)
```

### System Cleanup
```bash
# Remove all unused resources
docker system prune -af

# Remove unused volumes
docker volume prune -f

# Remove unused networks
docker network prune -f
```

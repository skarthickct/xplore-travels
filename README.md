# Xplore Travels Website

A Flask-based travel website with destination information and booking features.

## Features

- Browse travel destinations by categories (Beach, Adventure, Cultural, etc.)
- View detailed city information with budgets and recommendations
- Interactive travel planning with hotels and restaurants
- SQLite database with pre-populated travel data

## Architecture

### Tech Stack
- **Backend**: Flask (Python)
- **Database**: SQLite
- **Frontend**: HTML, CSS, JavaScript
- **Server**: Gunicorn (Production)
- **Containerization**: Docker (Multistage build)
- **CI/CD**: GitHub Actions

### Project Structure
```
app/
├── Dockerfile              # Optimized multistage build
├── app.py                 # Flask routes and application
├── dbms.py                # Database operations
├── requirements.txt       # Python dependencies
├── static/
│   ├── css/              # Stylesheets
│   ├── cities/           # City images
│   ├── activity/         # Activity images
│   └── GST Travels.db    # SQLite database
└── templates/            # HTML templates
```

## Quick Start

### Using Docker (Recommended)

```bash
# Build and run
cd app
docker build -t xplore-travels .
docker run -d -p 5000:5000 --name xplore-travels xplore-travels

# Access application
open http://localhost:5000
```

### Local Development

```bash
cd app
pip install -r requirements.txt
python app.py
```

## Database Schema

### Tables
- **city**: City information (cid, city)
- **tag**: Travel categories (tid, tag)
- **find**: City-tag relationships
- **destination**: Detailed destination info (budget, hotels, restaurants, etc.)

### Sample Data
- 22 cities across India
- 11 travel categories
- Complete destination details with pricing tiers

## API Endpoints

| Route | Description |
|-------|-------------|
| `/` | Home page |
| `/tag` | Browse travel categories |
| `/tag/<tag_id>` | Cities by category |
| `/city/<city>` | City details |
| `/<page>` | Static pages |

## Deployment

### GitHub Actions Workflow

Automatically deploys to EC2 on push to `main`:

1. **Build**: Creates optimized Docker image
2. **Push**: Uploads to Docker Hub
3. **Deploy**: Updates EC2 instance

### Required Secrets

```bash
DOCKER_USERNAME=your_dockerhub_username
DOCKER_PASSWORD=your_dockerhub_password
EC2_SSH_KEY=your_private_key_content
EC2_HOST=your_ec2_public_ip
```

### Manual Deployment

```bash
# On EC2 instance
docker pull username/xplore-travels:latest
docker stop xplore-travels || true
docker rm xplore-travels || true
docker run -d --name xplore-travels -p 5000:5000 --restart unless-stopped username/xplore-travels:latest
```

## Docker Optimization

### Image Size Reduction
- **Before**: 922MB (python:slim)
- **After**: 264MB (python:alpine)
- **Savings**: 71% smaller

### Security Features
- Non-root user execution
- Minimal attack surface
- Alpine Linux base

### Performance Features
- Gunicorn WSGI server
- Multi-worker configuration
- Production-ready setup

## Development

### Adding New Destinations

1. Update `dbms.py` with new city data
2. Add images to `static/cities/`
3. Run database initialization

### Customizing UI

1. Modify templates in `templates/`
2. Update styles in `static/css/`
3. Add new routes in `app.py`

## Monitoring

### Health Check
```bash
curl http://localhost:5000/
```

### Container Logs
```bash
docker logs xplore-travels
```

### Performance Metrics
- Response time: ~200ms
- Memory usage: ~50MB
- CPU usage: <5%

## Troubleshooting

### Common Issues

**Port already in use**
```bash
docker stop xplore-travels
```

**Database not found**
```bash
# Ensure GST Travels.db exists in static/
ls app/static/GST\ Travels.db
```

**Permission denied**
```bash
# Check file permissions
chmod +x app/start.sh
```

## Contributing

1. Fork the repository
2. Create feature branch
3. Make changes in `app/` directory
4. Test locally with Docker
5. Submit pull request

## License

MIT License - see LICENSE file for details.

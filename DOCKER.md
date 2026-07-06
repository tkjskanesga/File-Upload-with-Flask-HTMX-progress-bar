# Docker Deployment Guide

This guide explains how to build and run the File Upload Flask application using Docker.

## Prerequisites

- Docker installed on your system ([Download Docker](https://docs.docker.com/get-docker/))
- Docker Compose (optional, for easier management)

## Quick Start

### Build the Docker Image

```bash
docker build -t flask-file-upload .
```

### Run the Container

```bash
docker run -d \
  --name flask-upload-app \
  -p 5000:5000 \
  -v $(pwd)/storage:/root/storage \
  flask-file-upload
```

Access the application at: `http://localhost:5000/uploads`

## Environment Variables

You can customize the application behavior using environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `UPLOAD_FOLDER` | `/root/storage` | Directory path for uploaded files |
| `DATABASE_URI` | `sqlite:///data.sqlite` | SQLAlchemy database connection URI |
| `SECRET_KEY` | `random-secret-key-change-in-production` | Flask secret key for sessions |
| `FLASK_ENV` | `production` | Flask environment mode |
| `PORT` | `5000` | Application port |

### Running with Custom Environment Variables

```bash
docker run -d \
  --name flask-upload-app \
  -p 8080:8080 \
  -e PORT=8080 \
  -e SECRET_KEY=your-super-secret-key \
  -e UPLOAD_FOLDER=/root/storage \
  -v $(pwd)/storage:/root/storage \
  flask-file-upload
```

## Docker Compose (Recommended)

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  flask-app:
    build: .
    container_name: flask-upload-app
    ports:
      - "5000:5000"
    environment:
      - UPLOAD_FOLDER=/root/storage
      - SECRET_KEY=change-this-in-production
      - PORT=5000
    volumes:
      - ./storage:/root/storage
      - ./data.sqlite:/app/data.sqlite
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5000/uploads')"]
      interval: 30s
      timeout: 3s
      retries: 3
```

### Start with Docker Compose

```bash
# Start the application
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the application
docker-compose down
```

## Volume Mounting

### Persistent Storage

To persist uploaded files, mount a volume to the upload directory:

```bash
docker run -d \
  --name flask-upload-app \
  -p 5000:5000 \
  -v /path/on/host:/root/storage \
  flask-file-upload
```

### Database Persistence

To persist the SQLite database:

```bash
docker run -d \
  --name flask-upload-app \
  -p 5000:5000 \
  -v $(pwd)/storage:/root/storage \
  -v $(pwd)/data.sqlite:/app/data.sqlite \
  flask-file-upload
```

## Container Management

### View Running Containers

```bash
docker ps
```

### View Container Logs

```bash
docker logs flask-upload-app

# Follow logs in real-time
docker logs -f flask-upload-app
```

### Stop the Container

```bash
docker stop flask-upload-app
```

### Start the Container

```bash
docker start flask-upload-app
```

### Remove the Container

```bash
docker rm flask-upload-app
```

### Remove the Image

```bash
docker rmi flask-file-upload
```

## Health Check

The Docker image includes a health check that runs every 30 seconds. You can check the container health status:

```bash
docker inspect --format='{{.State.Health.Status}}' flask-upload-app
```

## Production Deployment

For production deployments, consider the following:

1. **Use a Reverse Proxy**: Run behind Nginx or Apache for better performance and security
2. **Set Strong SECRET_KEY**: Generate a secure random key
3. **Enable HTTPS**: Use TLS/SSL certificates
4. **Resource Limits**: Set memory and CPU limits
5. **Backup Strategy**: Regularly backup uploaded files and database

### Example with Resource Limits

```bash
docker run -d \
  --name flask-upload-app \
  -p 5000:5000 \
  -e SECRET_KEY=$(openssl rand -hex 32) \
  -v $(pwd)/storage:/root/storage \
  --memory="512m" \
  --cpus="1.0" \
  --restart=unless-stopped \
  flask-file-upload
```

## Troubleshooting

### Container Won't Start

Check logs for errors:
```bash
docker logs flask-upload-app
```

### Permission Issues

Ensure the upload directory has proper permissions:
```bash
chmod 755 storage/
```

### Port Already in Use

Change the host port mapping:
```bash
docker run -d -p 8080:5000 flask-file-upload
```

### Cannot Access Application

Verify the container is running and port is mapped correctly:
```bash
docker ps
docker port flask-upload-app
```

## Support

For issues or questions, please refer to:
- Application README: [README.md](README.md)
- Docker Documentation: [docs.docker.com](https://docs.docker.com)
- Flask Documentation: [flask.palletsprojects.com](https://flask.palletsprojects.com)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

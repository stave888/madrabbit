# MadRabbit Docker Deployment

This directory contains Docker configurations for deploying the MadRabbit vulnerability lab platform.

## Quick Start

### Using Docker Compose (Recommended)

The easiest way to deploy MadRabbit with all dependencies:

```bash
# From the project root directory
cd docker
docker-compose up -d
```

This will:
- Start MySQL 8.0 with automatic database initialization
- Build and run the MadRabbit application
- Set up networking between containers

Access the application at: http://localhost:8080/login.html

### Using Dockerfile Only

If you have an existing MySQL instance:

```bash
# Build the image
docker build -t madrabbit:latest -f docker/Dockerfile .

# Run the container
docker run -d \
  -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://YOUR_MYSQL_HOST:3306/madrabbit?useSSL=false&serverTimezone=UTC&characterEncoding=utf8 \
  -e SPRING_DATASOURCE_USERNAME=YOUR_DB_USERNAME \
  -e SPRING_DATASOURCE_PASSWORD=YOUR_DB_PASSWORD \
  --name madrabbit-app \
  madrabbit:latest
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SPRING_DATASOURCE_URL` | MySQL JDBC connection string | `jdbc:mysql://mysql:3306/madrabbit` |
| `SPRING_DATASOURCE_USERNAME` | Database username | `madrabbit` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | `madrabbit123` |
| `JAVA_OPTS` | Additional JVM options | Preconfigured for deserialization |

### Ports

- **8080**: Application HTTP port
- **3306**: MySQL database port (docker-compose only)

## Default Test Accounts

| Username | Password | Role |
|----------|----------|------|
| admin | 123456 | ADMIN |
| jack | 123456 | LEARNER |
| lucy | 123456 | LEARNER |
| tom | 123456 | LEARNER |

## Management Commands

```bash
# View logs
docker-compose logs -f app

# Stop services
docker-compose down

# Stop and remove volumes (⚠️ deletes database data)
docker-compose down -v

# Rebuild after code changes
docker-compose up -d --build

# Check service status
docker-compose ps
```

## Architecture

The Docker setup uses:
- **Multi-stage build**: Separates Maven build from runtime for smaller images
- **Health checks**: Ensures MySQL is ready before starting the app
- **Volume persistence**: Database data survives container restarts
- **Network isolation**: Dedicated bridge network for service communication

## Security Warning

> **WARNING**: This platform contains intentionally vulnerable code for educational purposes.
> 
> - **DO NOT** expose to the public internet
> - **DO NOT** deploy in production environments
> - Use only in isolated lab/training environments
> - Default passwords should be changed for any non-local deployment

## Troubleshooting

### Application won't start
- Check MySQL is healthy: `docker-compose logs mysql`
- Verify database initialization: `docker-compose exec mysql mysql -u madrabbit -pmadrabbit123 madrabbit -e "SHOW TABLES;"`

### Connection refused
- Ensure MySQL service is fully started (health check passes)
- Check network: `docker network inspect docker_madrabbit-network`

### Build failures
- Clear Maven cache: `docker-compose build --no-cache`
- Check disk space: `docker system df`

## Resource Requirements

- **CPU**: 2+ cores recommended
- **Memory**: 2GB minimum (4GB recommended)
- **Disk**: ~1GB for images + database storage

## Development

To rebuild only the application after code changes:

```bash
docker-compose up -d --build app
```

For faster iteration, consider using `mvn spring-boot:run` locally instead of rebuilding the Docker image.

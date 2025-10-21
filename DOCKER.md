# Docker Setup Guide

This guide covers running Talk to the City using Docker and Docker Compose.

## Quick Start

### Prerequisites

- Docker Desktop or Docker Engine + Docker Compose
- Copy and configure environment variables

### Setup

1. **Copy environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your actual configuration values
   ```

2. **Start all services:**
   ```bash
   # Production build
   docker-compose up --build

   # Development mode with hot reloading
   docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build
   ```

3. **Access the application:**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8080
   - Python LLM service: http://localhost:8000
   - Redis: localhost:6379

## Architecture

The Docker setup includes these services:

- **next-client**: React/Next.js frontend (port 3000)
- **express-server**: Node.js/Express backend API (port 8080)
- **pyserver**: Python/FastAPI LLM processing service (port 8000)
- **redis**: Redis for job queue management (port 6379)

## Configuration

### Environment Variables

All required environment variables are documented in `.env.example`. Key variables include:

- **Firebase credentials**: For authentication and database
- **Google Cloud Storage**: For report file storage
- **OpenAI API key**: For LLM processing
- **Redis configuration**: For job queue management

### Service Communication

- `next-client` → `express-server` (API calls)
- `express-server` → `pyserver` (LLM job processing)
- `express-server` → `redis` (job queue management)

## Development vs Production

### Development Mode

For active development with hot reloading:

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build
```

Features:
- Hot reloading for all services
- Source code mounted as volumes
- Development dependencies included
- Detailed logging enabled

### Production Mode

For production deployment:

```bash
docker-compose up --build
```

Features:
- Optimized builds with multi-stage Dockerfiles
- Minimal production images
- Security hardening
- Health checks enabled

## Common Commands

### Build and Start Services
```bash
# Production
docker-compose up --build

# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --build

# Detached mode
docker-compose up -d --build
```

### Individual Service Management
```bash
# Start specific service
docker-compose up next-client

# View logs
docker-compose logs -f express-server

# Execute commands in running container
docker-compose exec express-server npm test
docker-compose exec pyserver pytest
```

### Database and Storage
```bash
# View Redis data
docker-compose exec redis redis-cli

# Backup Redis data (data persists in named volume)
docker-compose exec redis redis-cli save
```

### Cleanup
```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Remove all images
docker-compose down --rmi all
```

## Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 3000, 8000, 8080, and 6379 are available
2. **Environment variables**: Verify `.env` file exists and contains valid values
3. **Service dependencies**: Services start in dependency order with health checks

### Health Checks

All services include health checks:
- **next-client**: HTTP check on port 3000
- **express-server**: HTTP check on `/health` endpoint
- **pyserver**: HTTP check on `/health` endpoint
- **redis**: Redis ping command

### Viewing Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f express-server

# Follow logs with timestamps
docker-compose logs -f -t
```

### Debugging

```bash
# Connect to running container
docker-compose exec express-server bash
docker-compose exec pyserver bash

# Check service status
docker-compose ps

# Restart specific service
docker-compose restart express-server
```

## Production Deployment

For production deployment, consider:

1. **Secrets management**: Use Docker secrets or external secret management
2. **Load balancing**: Use nginx or a load balancer in front of services
3. **Monitoring**: Add monitoring and logging solutions
4. **Backup**: Regular backup of Redis data and application data
5. **Security**: Review and harden security configurations

### Example Production Override

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'
services:
  next-client:
    restart: unless-stopped
    environment:
      - NODE_ENV=production
  
  express-server:
    restart: unless-stopped
    environment:
      - NODE_ENV=production
  
  redis:
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
```

Then run:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Performance Optimization

### Development
- Use cached volume mounts for better performance on macOS/Windows
- Limit service rebuilds to only changed services
- Use `.dockerignore` files to exclude unnecessary files

### Production
- Multi-stage builds minimize image size
- Non-root users for security
- Health checks ensure service reliability
- Persistent volumes for data retention
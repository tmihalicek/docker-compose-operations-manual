# Docker Compose Operations Manual

## Table of Contents
1. [Introduction](#introduction)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Concepts](#basic-concepts)
4. [File Structure and Syntax](#file-structure-and-syntax)
5. [Core Commands](#core-commands)
6. [Service Configuration](#service-configuration)
7. [Networking](#networking)
8. [Storage and Volumes](#storage-and-volumes)
9. [Environment Management](#environment-management)
10. [Monitoring and Logging](#monitoring-and-logging)
11. [Security Best Practices](#security-best-practices)
12. [Troubleshooting](#troubleshooting)
13. [Production Deployment](#production-deployment)
14. [Common Patterns and Examples](#common-patterns-and-examples)

## Introduction

Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure application services, networks, and volumes, allowing you to manage complex applications with a single command.

### Key Benefits
- Simplified multi-container orchestration
- Reproducible development environments
- Easy service dependency management
- Version-controlled infrastructure configuration

## Installation and Setup

### Prerequisites
- Docker Engine installed and running
- Basic understanding of containerization concepts
- Command line interface access

### Installation Methods

#### Linux (Debian)
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the latest version
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
docker compose version
```

#### macOS
```bash
# Using Homebrew (Docker Desktop)
brew install docker-desktop
```
### Verification
```bash
docker compose --help
docker compose version
```

## Basic Concepts

### Key Components
- **Services**: Individual containers that make up your application
- **Networks**: Communication channels between services
- **Volumes**: Persistent data storage shared between containers
- **Images**: Templates used to create containers
- **Environment**: Configuration variables and settings

### Compose File Hierarchy
```
project-root/
├── compose.yml          # Main compose file
├── .env                        # Environment variables
└── services/
    ├── web/
    │   └── Dockerfile
    └── database/
        └── init.sql
```

## File Structure and Syntax

### Basic compose.yaml Structure
```yaml
services:
  service-name:
    # Service configuration
    
networks:
  # Network definitions
  
volumes:
  # Volume definitions
  
configs:
  # Configuration definitions
  
secrets:
  # Secret definitions
```

### Essential Service Properties
```yaml
services:
  web:
    image: nginx:alpine                   # Pre-built image
    build: ./web                          # Build from Dockerfile
    container_name: my-web-container      # Custom container name
    ports:
      - "8080:80"                         # Port mapping
    volumes:
      - ./html:/usr/share/nginx/html      # Volume mount
    environment:
      - ENV_VAR=value                     # Environment variables
    depends_on:
      - database                          # Service dependencies
    restart: unless-stopped               # Restart policy
    networks:
      - frontend                          # Network assignment
```

## Core Commands

### Project Lifecycle
```bash
# Start services (build if necessary)
docker compose up

# Start in detached mode
docker compose up -d

# Start specific services
docker compose up web database

# Stop services (preserves containers)
docker compose stop

# Stop and remove containers
docker compose down

# Stop and remove containers, networks, volumes
docker compose down -v

# Restart services
docker compose restart
```

### Building and Images
```bash
# Build or rebuild services
docker compose build

# Build specific service
docker compose build web

# Build without cache
docker compose build --no-cache

# Pull latest images
docker compose pull
```

### Service Management
```bash
# View running services
docker compose ps

# View all services (including stopped)
docker compose ps -a

# View service logs
docker compose logs

# Follow logs in real-time
docker compose logs -f

# View logs for specific service
docker compose logs web

# Execute commands in running container
docker compose exec web bash

# Run one-off commands
docker compose run web python manage.py migrate
```

### Scaling Services
```bash
# Scale service to multiple instances
docker compose up --scale web=3

# Scale multiple services
docker compose up --scale web=3 --scale worker=2
```

## Service Configuration

### Image Sources
```yaml
services:
  # Using pre-built image
  web:
    image: nginx:1.21-alpine
    
  # Building from Dockerfile
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
        
  # Building with custom image name
  worker:
    build: ./worker
    image: myapp-worker:latest
```

### Port Configuration
```yaml
services:
  web:
    ports:
      - "80:80"           # host:container
      - "443:443"
      - "127.0.0.1:8080:80"  # bind to specific interface
      - "3000-3005:3000-3005"  # port range
    expose:
      - "9000"            # internal port (no host mapping)
```

### Environment Variables
```yaml
services:
  app:
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    env_file:
      - .env
      - .env.local
```

### Health Checks
```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Resource Limits
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
```

## Networking

### Network Types
```yaml
networks:
  # Bridge network (default)
  frontend:
    driver: bridge
    
  # Custom bridge with subnet
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          
  # External network
  shared:
    external: true
    name: shared-network
```

### Service Networking
```yaml
services:
  web:
    networks:
      - frontend
      - backend
    depends_on:
      - api
      - database
      
  api:
    networks:
      backend:
        aliases:
          - api-server
        ipv4_address: 172.20.0.10
```

### Network Aliases and Service Discovery
```yaml
services:
  database:
    image: postgres:13
    networks:
      backend:
        aliases:
          - db
          - postgres-server
  
  app:
    image: myapp
    networks:
      - backend
    environment:
      - DB_HOST=db  # Can use alias instead of service name
```

## Storage and Volumes

### Volume Types
```yaml
volumes:
  # Named volume
  database_data:
    driver: local
    
  # External volume
  shared_data:
    external: true
    
  # NFS volume
  nfs_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/path/to/share"
```

### Volume Mounting
```yaml
services:
  database:
    image: postgres:13
    volumes:
      # Named volume
      - database_data:/var/lib/postgresql/data
      # Bind mount
      - ./backups:/backups
      # Anonymous volume
      - /var/log/postgres
      # Read-only mount
      - ./config:/etc/postgres:ro
    
  web:
    image: nginx
    volumes:
      # Bind mount with specific options
      - type: bind
        source: ./html
        target: /usr/share/nginx/html
        read_only: true
```

### Backup and Restore Strategies
```yaml
services:
  backup:
    image: alpine
    volumes:
      - database_data:/data:ro
      - ./backups:/backup
    command: |
      sh -c "
        tar czf /backup/backup-$$(date +%Y%m%d-%H%M%S).tar.gz -C /data .
        find /backup -name '*.tar.gz' -mtime +7 -delete
      "
```

## Environment Management

### Environment Files
```bash
# .env file
NODE_ENV=production
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
REDIS_URL=redis://localhost:6379
API_KEY=your-secret-key
```

## Monitoring and Logging

### Centralized Logging
```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        
  web:
    image: nginx
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://192.168.1.100:514"
```

## Security Best Practices

### Secret Management
```yaml
services:
  app:
    image: myapp
    secrets:
      - db_password
      - api_key
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      
secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true
    name: myapp_api_key
```

### Network Isolation
```yaml
networks:
  frontend:
    driver: bridge
    internal: false  # Can access external networks
    
  backend:
    driver: bridge
    internal: true   # Isolated from external networks
    
services:
  web:
    networks:
      - frontend
      
  api:
    networks:
      - frontend
      - backend
      
  database:
    networks:
      - backend  # Only accessible from backend network
```

### User and Permissions
```yaml
services:
  app:
    image: myapp
    user: "1000:1000"  # Run as non-root user
    read_only: true    # Read-only root filesystem
    tmpfs:
      - /tmp
      - /var/tmp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

### Resource Limits
```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
    security_opt:
      - no-new-privileges:true
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

## Troubleshooting

### Common Issues and Solutions

#### Container Startup Failures
```bash
# Check service status
docker compose ps

# View detailed logs
docker compose logs service-name

# Inspect container configuration
docker compose config

# Debug container interactively
docker compose run --rm service-name sh
```

#### Network Connectivity Issues
```bash
# List networks
docker network ls

# Inspect network configuration
docker network inspect project_default

# Test connectivity between services
docker compose exec service1 ping service2

# Check port bindings
docker compose port service-name 80
```

#### Volume and Permissions Issues
```bash
# Check volume mounts
docker compose exec service-name mount | grep volume

# Fix permission issues
docker compose exec service-name chown -R user:group /path

# Inspect volume details
docker volume inspect project_volume-name
```

#### Resource Constraints
```bash
# Monitor resource usage
docker stats

# Check system resources
docker system df

# Clean up unused resources
docker system prune -f
```

### Debugging Techniques
```yaml
services:
  debug:
    image: alpine
    volumes:
      - .:/workspace
    working_dir: /workspace
    command: sh -c "while true; do sleep 3600; done"
    
  app:
    image: myapp
    volumes:
      - .:/app
    command: |
      sh -c "
        echo 'Starting application...'
        set -x  # Enable debug output
        exec npm start
      "
```

## Production Deployment

### Production Compose Configuration
```yaml
services:
  web:
    image: myapp:${TAG:-latest}
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    env_file:
      - .env.prod
    volumes:
      - static_files:/app/static
      - ./ssl:/etc/ssl:ro
    depends_on:
      - database
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
        reservations:
          memory: 512M
          cpus: '0.5'

  database:
    image: postgres:13-alpine
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 5s
      retries: 3

  redis:
    image: redis:6-alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  static_files:
    driver: local

networks:
  default:
    driver: bridge
```

## Common Patterns and Examples

### LAMP Stack
```yaml
services:
  apache:
    image: php:8.0-apache
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

### Node.js with MongoDB
```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/myapp
    depends_on:
      - mongodb
    volumes:
      - ./uploads:/app/uploads

  mongodb:
    image: mongo:5.0
    volumes:
      - mongodb_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password

volumes:
  mongodb_data:
```

---

*This manual provides comprehensive guidance for Docker Compose operations. For the latest features and updates, always refer to the official Docker Compose documentation.*

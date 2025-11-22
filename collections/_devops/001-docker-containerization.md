---
layout: post
title: "Docker & Containerization: Building, Deploying, and Scaling Apps"
author: "Liberom"
date: 2025-01-09
category: "DevOps"
toc: true
excerpt: "Master Docker fundamentals: containerization, images, Compose, and deployment best practices."
---

# Docker & Containerization: Building, Deploying, and Scaling Apps

Docker revolutionized deployment by standardizing applications into portable containers. Understanding containers is essential for modern DevOps.

## Docker Fundamentals

### What is Docker?

Docker packages your application and all its dependencies (OS, runtime, libraries) into a single **image**. This image can run identically on any system.

**Key Concepts**:
- **Image**: Blueprint for a container (immutable)
- **Container**: Running instance of an image (mutable)
- **Dockerfile**: Instructions to build an image
- **Registry**: Repository of images (Docker Hub, ECR, etc.)

### Installing Docker

```bash
# macOS
brew install docker docker-compose

# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Verify installation
docker --version
docker run hello-world
```

## Dockerfiles

### Basic Dockerfile Structure

```dockerfile
# FROM - base image
FROM ruby:3.2-slim

# WORKDIR - working directory inside container
WORKDIR /app

# RUN - execute commands during build
RUN apt-get update && apt-get install -y \
  build-essential \
  postgresql-client

# COPY - copy files from host to container
COPY Gemfile Gemfile.lock ./
RUN bundle install --frozen

# COPY - copy application code
COPY . .

# EXPOSE - document ports
EXPOSE 3000

# CMD - default command when container starts
CMD ["rails", "server", "-b", "0.0.0.0"]
```

### Multi-Stage Builds (Smaller Images)

```dockerfile
# Build stage
FROM ruby:3.2 as builder
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Runtime stage
FROM ruby:3.2-slim
WORKDIR /app
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY . .
EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
```

### Environment Variables

```dockerfile
# In Dockerfile
ENV RAILS_ENV=production
ENV NODE_ENV=production

# Build-time arguments
ARG BUNDLE_WITHOUT=development:test
RUN bundle install --without=$BUNDLE_WITHOUT

# Usage: docker build --build-arg BUNDLE_WITHOUT=development:test .
```

## Building and Running Containers

### Building Images

```bash
# Basic build
docker build -t myapp:1.0 .

# Build with arguments
docker build -t myapp:1.0 --build-arg RUBY_VERSION=3.2 .

# Build with no cache
docker build -t myapp:1.0 --no-cache .

# Tag image for registry
docker tag myapp:1.0 myregistry/myapp:1.0
docker push myregistry/myapp:1.0
```

### Running Containers

```bash
# Basic run
docker run myapp:1.0

# Interactive terminal
docker run -it myapp:1.0 /bin/bash

# Port mapping
docker run -p 3000:3000 myapp:1.0

# Environment variables
docker run -e RAILS_ENV=production -e DATABASE_URL=postgres://... myapp:1.0

# Volume mounting
docker run -v /host/path:/container/path myapp:1.0

# Background (detached)
docker run -d -p 3000:3000 --name myapp myapp:1.0

# Container management
docker ps                          # Running containers
docker ps -a                       # All containers
docker logs myapp                  # View logs
docker exec -it myapp /bin/bash    # Run command in container
docker stop myapp                  # Stop container
docker rm myapp                    # Remove container
```

## Docker Compose

### Multi-Container Applications

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - RAILS_ENV=development
      - DATABASE_URL=postgres://postgres:password@db:5432/myapp_dev
    depends_on:
      - db
      - redis
    volumes:
      - .:/app
    command: rails server -b 0.0.0.0

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Using Docker Compose

```bash
# Start all services
docker-compose up

# Background
docker-compose up -d

# View logs
docker-compose logs -f web

# Run migration
docker-compose exec web rails db:migrate

# Run bash in container
docker-compose exec web /bin/bash

# Rebuild images
docker-compose up --build

# Stop services
docker-compose stop

# Remove everything
docker-compose down

# Remove with volumes
docker-compose down -v
```

## Best Practices

### Dockerfile Best Practices

```dockerfile
# GOOD - Minimize layers, combine commands
FROM ruby:3.2-slim
RUN apt-get update && apt-get install -y \
  build-essential \
  postgresql-client && \
  rm -rf /var/lib/apt/lists/*

# BAD - Creates unnecessary layers
FROM ruby:3.2-slim
RUN apt-get update
RUN apt-get install -y build-essential
RUN apt-get install -y postgresql-client

# GOOD - Use .dockerignore
# .dockerignore
.git
.gitignore
node_modules
tmp
log
coverage
.env
.env.local

# GOOD - Non-root user for security
FROM ruby:3.2-slim
RUN useradd -m -u 1000 app
USER app

# GOOD - Health checks
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# GOOD - Explicit base image version
FROM ruby:3.2.0-slim-bullseye

# BAD - Using 'latest' tag
FROM ruby:latest
```

### Production-Ready Dockerfile

```dockerfile
FROM ruby:3.2-slim as builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
  build-essential \
  git \
  postgresql-client \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install --deployment --without development:test

# Runtime stage
FROM ruby:3.2-slim

RUN apt-get update && apt-get install -y \
  postgresql-client \
  && rm -rf /var/lib/apt/lists/*

RUN useradd -m -u 1000 app

WORKDIR /app
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --chown=app:app . .

USER app
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

## Networking and Data

### Container Networking

```bash
# Create custom network
docker network create mynetwork

# Run containers on network
docker run --network mynetwork --name web myapp:1.0
docker run --network mynetwork --name db postgres:15

# Containers can communicate using service names
# Inside web container: connect to db using hostname 'db'
```

### Volumes and Persistence

```bash
# Named volume
docker run -v mydata:/app/data myapp:1.0

# Bind mount
docker run -v /host/path:/app/data myapp:1.0

# Volume inspection
docker volume ls
docker volume inspect mydata
docker volume rm mydata

# Backup volume
docker run --rm -v mydata:/data -v /host/backup:/backup \
  ubuntu tar czf /backup/backup.tar.gz /data
```

## Debugging and Monitoring

### Viewing Logs

```bash
# View container logs
docker logs myapp

# Follow logs (tail)
docker logs -f myapp

# Last 100 lines
docker logs --tail 100 myapp

# With timestamps
docker logs -t myapp

# Docker compose logs
docker-compose logs -f web
```

### Inspecting Containers

```bash
# Container details
docker inspect myapp

# Resource usage
docker stats

# Network information
docker network inspect mynetwork

# Processes in container
docker top myapp
```

### Shell Access and Debugging

```bash
# Interactive shell
docker exec -it myapp /bin/bash

# Run command
docker exec myapp ps aux
docker exec myapp rails console
docker exec myapp rails db:migrate

# Copy files
docker cp myapp:/app/file.txt ./
docker cp ./file.txt myapp:/app/
```

## Common Mistakes

```dockerfile
# BAD - Running as root
FROM ubuntu:22.04
WORKDIR /app
COPY . .
CMD ["./app"]

# GOOD - Non-root user
FROM ubuntu:22.04
RUN useradd -m app
USER app
WORKDIR /app
COPY --chown=app:app . .
CMD ["./app"]

# BAD - Single layer, large image
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y ruby && \
    apt-get install -y build-essential && \
    apt-get install -y postgresql-client && \
    app-specific-setup.sh

# GOOD - Cleaned up layers
FROM ruby:3.2-slim
RUN apt-get update && apt-get install -y \
  build-essential \
  postgresql-client && \
  rm -rf /var/lib/apt/lists/*

# BAD - No health check
docker run -d myapp:1.0

# GOOD - With health check
docker run -d --health-cmd='curl -f http://localhost:3000 || exit 1' myapp:1.0
```

## Conclusion

Docker containerization principles:
- Keep images small using multi-stage builds
- Always specify base image versions (avoid latest)
- Run containers as non-root users
- Use Docker Compose for local development
- Add health checks for production
- Implement proper logging and monitoring
- Follow security best practices


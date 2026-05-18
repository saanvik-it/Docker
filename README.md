# 🐳 Docker — Complete Guide

> A comprehensive reference covering Docker architecture, commands, Dockerfiles, and interview Q&A.

---

## Table of Contents

1. [What is Docker?](#what-is-docker)
2. [Docker Architecture](#docker-architecture)
3. [Core Concepts](#core-concepts)
4. [Installation](#installation)
5. [Essential Docker Commands](#essential-docker-commands)
6. [Dockerfile — Complete Reference](#dockerfile--complete-reference)
7. [Docker Compose](#docker-compose)
8. [Networking in Docker](#networking-in-docker)
9. [Docker Volumes](#docker-volumes)
10. [Interview Questions & Answers](#interview-questions--answers)

---

## What is Docker?

Docker is an open-source **containerization platform** that allows developers to package applications and their dependencies into lightweight, portable containers. These containers run consistently across any environment — development, staging, or production.

**Key Benefits:**
- Consistent environments across all stages
- Lightweight compared to virtual machines
- Fast startup times (milliseconds vs minutes)
- Easy scaling and orchestration
- Isolation between applications

---

## Docker Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Docker Client                        │
│              (docker build / run / pull / push)             │
└──────────────────────────┬──────────────────────────────────┘
                           │  REST API
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      Docker Daemon (dockerd)                │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐   │
│  │   Images    │  │ Containers  │  │    Networks /    │   │
│  │  (stored)   │  │  (running)  │  │    Volumes       │   │
│  └─────────────┘  └─────────────┘  └──────────────────┘   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     Container Runtime                       │
│              (containerd → runc → Linux Kernel)             │
│                                                             │
│  Namespaces (PID, NET, MNT, UTS, IPC, USER)                 │
│  cgroups (CPU, Memory, I/O limits)                          │
│  Union File System (OverlayFS)                              │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      Docker Registry                        │
│          Docker Hub / ECR / GCR / Private Registry          │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Components

| Component | Description |
|---|---|
| **Docker Client** | CLI tool that sends commands to the Docker daemon via REST API |
| **Docker Daemon (dockerd)** | Background service managing images, containers, networks, volumes |
| **containerd** | High-level container runtime managing lifecycle of containers |
| **runc** | Low-level OCI runtime that actually creates containers |
| **Docker Registry** | Storage for Docker images (Docker Hub is the default public registry) |
| **Namespaces** | Isolate process trees, networks, mounts, users per container |
| **cgroups** | Limit and account for resource usage (CPU, RAM, disk I/O) |
| **OverlayFS** | Union file system enabling layered images |

---

## Core Concepts

### Image vs Container

```
Dockerfile  ──build──►  Image  ──run──►  Container
(blueprint)             (template)        (running instance)
```

- **Image** — Read-only template with layers. Built from a Dockerfile.
- **Container** — A running instance of an image. Has its own writable layer.
- **Layer** — Each instruction in a Dockerfile creates a new layer (cached).

### Image Layers

```
┌──────────────────────────┐  ◄── Writable Container Layer
├──────────────────────────┤
│   COPY app/ /app         │  ◄── Layer 4
├──────────────────────────┤
│   RUN pip install -r ... │  ◄── Layer 3
├──────────────────────────┤
│   RUN apt-get update     │  ◄── Layer 2
├──────────────────────────┤
│   FROM python:3.11-slim  │  ◄── Base Layer 1
└──────────────────────────┘
```

---

## Installation

### Linux (Ubuntu/Debian)

```bash
# Update packages
sudo apt-get update

# Install dependencies
sudo apt-get install ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repo and install
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
```

### macOS / Windows
Download and install **Docker Desktop** from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

### Verify Installation

```bash
docker --version
docker info
docker run hello-world
```

---

## Essential Docker Commands

### Image Commands

```bash
# Pull an image from Docker Hub
docker pull nginx:latest
docker pull python:3.11-slim

# List all local images
docker images
docker image ls

# Build an image from Dockerfile
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .

# Tag an image
docker tag myapp:1.0 myrepo/myapp:1.0

# Push image to registry
docker push myrepo/myapp:1.0

# Remove an image
docker rmi myapp:1.0
docker image rm myapp:1.0

# Remove all unused images
docker image prune -a

# Inspect image details
docker inspect nginx:latest

# View image history/layers
docker history nginx:latest
```

### Container Commands

```bash
# Run a container
docker run nginx                              # foreground
docker run -d nginx                           # detached (background)
docker run -d -p 8080:80 nginx               # port mapping host:container
docker run -d --name webserver nginx         # with name
docker run -it ubuntu bash                   # interactive terminal
docker run --rm ubuntu echo "hello"          # remove after exit

# Run with env variables
docker run -d -e DB_HOST=localhost -e DB_PORT=5432 myapp

# Run with volume mount
docker run -d -v /host/path:/container/path nginx
docker run -d -v myvolume:/data nginx

# Run with resource limits
docker run -d --memory="512m" --cpus="1.0" myapp

# List containers
docker ps                    # running containers
docker ps -a                 # all containers (including stopped)
docker ps -q                 # only container IDs

# Start / Stop / Restart
docker start container_name
docker stop container_name
docker restart container_name

# Kill a container immediately
docker kill container_name

# Remove a container
docker rm container_name
docker rm -f container_name       # force remove running container

# Remove all stopped containers
docker container prune

# Execute command in running container
docker exec -it container_name bash
docker exec container_name ls /app

# View container logs
docker logs container_name
docker logs -f container_name      # follow/stream logs
docker logs --tail 100 container_name

# Copy files between host and container
docker cp ./file.txt container_name:/app/file.txt
docker cp container_name:/app/output.txt ./output.txt

# View container stats (live)
docker stats
docker stats container_name

# Inspect container details
docker inspect container_name

# View running processes
docker top container_name
```

### Volume Commands

```bash
# Create a volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect myvolume

# Remove a volume
docker volume rm myvolume

# Remove all unused volumes
docker volume prune
```

### Network Commands

```bash
# List networks
docker network ls

# Create a network
docker network create mynetwork
docker network create --driver bridge mynetwork

# Connect container to network
docker network connect mynetwork container_name

# Disconnect container
docker network disconnect mynetwork container_name

# Inspect network
docker network inspect mynetwork

# Remove a network
docker network rm mynetwork
```

### System Commands

```bash
# Show system-wide info
docker info

# Show disk usage
docker system df

# Remove all unused resources (images, containers, networks, volumes)
docker system prune
docker system prune -a --volumes     # aggressive cleanup

# View Docker events in real time
docker events

# Login to Docker Hub
docker login
docker login registry.example.com
```

---

## Dockerfile — Complete Reference

### Basic Syntax

```dockerfile
# Comment
INSTRUCTION arguments
```

### All Instructions Explained

```dockerfile
# ─────────────────────────────────────────────
# 1. FROM — Base image (must be first instruction)
# ─────────────────────────────────────────────
FROM ubuntu:22.04
FROM python:3.11-slim AS builder     # multi-stage build alias
FROM scratch                          # empty base image

# ─────────────────────────────────────────────
# 2. LABEL — Metadata
# ─────────────────────────────────────────────
LABEL maintainer="dev@example.com"
LABEL version="1.0" description="My App"

# ─────────────────────────────────────────────
# 3. ARG — Build-time variables (not in final image)
# ─────────────────────────────────────────────
ARG APP_VERSION=1.0
ARG ENVIRONMENT=production

# ─────────────────────────────────────────────
# 4. ENV — Environment variables (persisted in image)
# ─────────────────────────────────────────────
ENV APP_HOME=/app
ENV PORT=8080 DEBUG=false

# ─────────────────────────────────────────────
# 5. RUN — Execute commands during build
# ─────────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*    # Clean up in same layer!

# Shell form vs Exec form
RUN echo "hello"                      # shell form
RUN ["echo", "hello"]                 # exec form (preferred)

# ─────────────────────────────────────────────
# 6. WORKDIR — Set working directory
# ─────────────────────────────────────────────
WORKDIR /app                          # creates dir if not exists

# ─────────────────────────────────────────────
# 7. COPY — Copy files from build context
# ─────────────────────────────────────────────
COPY . .                              # copy all to WORKDIR
COPY package*.json ./                 # copy specific files
COPY --from=builder /app/dist ./dist  # copy from another stage

# ─────────────────────────────────────────────
# 8. ADD — Like COPY but supports URLs and auto-extracts tar
# ─────────────────────────────────────────────
ADD https://example.com/file.tar.gz /tmp/   # from URL
ADD archive.tar.gz /app/                     # auto-extracts tar
# Prefer COPY over ADD unless you need these features

# ─────────────────────────────────────────────
# 9. EXPOSE — Document the port (does NOT publish it)
# ─────────────────────────────────────────────
EXPOSE 8080
EXPOSE 8080/tcp 8081/udp

# ─────────────────────────────────────────────
# 10. VOLUME — Create a mount point
# ─────────────────────────────────────────────
VOLUME ["/data", "/logs"]

# ─────────────────────────────────────────────
# 11. USER — Set runtime user (security best practice)
# ─────────────────────────────────────────────
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
USER appuser

# ─────────────────────────────────────────────
# 12. CMD — Default command (can be overridden at runtime)
# ─────────────────────────────────────────────
CMD ["python", "app.py"]              # exec form (preferred)
CMD python app.py                     # shell form
# Only the last CMD takes effect

# ─────────────────────────────────────────────
# 13. ENTRYPOINT — Fixed command (CMD becomes default args)
# ─────────────────────────────────────────────
ENTRYPOINT ["python", "app.py"]
# With both: ENTRYPOINT is the executable, CMD provides default args
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]

# ─────────────────────────────────────────────
# 14. HEALTHCHECK — Monitor container health
# ─────────────────────────────────────────────
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# ─────────────────────────────────────────────
# 15. ONBUILD — Trigger for child images
# ─────────────────────────────────────────────
ONBUILD COPY . /app
ONBUILD RUN pip install -r requirements.txt

# ─────────────────────────────────────────────
# 16. STOPSIGNAL — Signal to stop the container
# ─────────────────────────────────────────────
STOPSIGNAL SIGTERM

# ─────────────────────────────────────────────
# 17. SHELL — Override default shell for RUN
# ─────────────────────────────────────────────
SHELL ["/bin/bash", "-c"]
```

### Real-World Dockerfile Examples

#### Python Flask Application

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Copy requirements first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m appuser && chown -R appuser /app
USER appuser

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost:5000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

#### Node.js Application

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18-alpine

WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

#### Multi-Stage Build (Go Application)

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

# Final stage — minimal image
FROM scratch
COPY --from=builder /build/app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```

### .dockerignore

```
# .dockerignore — Exclude files from build context
node_modules/
.git/
.gitignore
*.log
*.md
.env
.env.*
dist/
coverage/
__pycache__/
*.pyc
.DS_Store
Dockerfile*
docker-compose*
```

---

## Docker Compose

### docker-compose.yml — Full Example

```yaml
version: '3.9'

services:
  # Web Application
  web:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - APP_VERSION=1.0
    image: myapp:latest
    container_name: myapp_web
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - REDIS_URL=redis://cache:6379
    env_file:
      - .env
    volumes:
      - ./logs:/app/logs
      - static_files:/app/static
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    container_name: myapp_db
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Redis Cache
  cache:
    image: redis:7-alpine
    container_name: myapp_cache
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - backend
    restart: unless-stopped

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: myapp_nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - static_files:/usr/share/nginx/html/static:ro
    depends_on:
      - web
    networks:
      - frontend
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  static_files:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true     # no external access
```

### Docker Compose Commands

```bash
# Start all services
docker compose up
docker compose up -d                    # detached
docker compose up --build               # rebuild images first

# Stop services
docker compose down
docker compose down -v                  # also remove volumes
docker compose down --rmi all           # also remove images

# View logs
docker compose logs
docker compose logs -f web              # follow specific service

# Scale a service
docker compose up --scale web=3

# Run one-off command
docker compose run web python manage.py migrate
docker compose exec web bash

# View running services
docker compose ps

# Restart a service
docker compose restart web

# Pull latest images
docker compose pull
```

---

## Networking in Docker

### Network Drivers

| Driver | Description | Use Case |
|---|---|---|
| **bridge** | Default, isolated network on same host | Single-host apps |
| **host** | Container shares host network stack | Performance-critical apps |
| **none** | No networking | Total isolation |
| **overlay** | Multi-host networking | Docker Swarm / distributed |
| **macvlan** | Container gets its own MAC address | Legacy apps needing direct network access |

### Network Examples

```bash
# Create custom bridge network
docker network create --driver bridge --subnet 172.18.0.0/16 mynet

# Run containers on same network (can communicate by name)
docker run -d --name db --network mynet postgres
docker run -d --name web --network mynet -e DB_HOST=db myapp

# Container DNS — containers on same network resolve by name
# 'web' container can reach 'db' using hostname 'db'
```

---

## Docker Volumes

### Volume Types

```bash
# Named volume (managed by Docker)
docker run -v myvolume:/data nginx

# Bind mount (host path)
docker run -v /host/path:/container/path nginx

# tmpfs mount (in-memory, not persisted)
docker run --tmpfs /tmp nginx

# Read-only mount
docker run -v myvolume:/data:ro nginx
```

---

## Interview Questions & Answers

### Beginner Level

---

**Q1. What is Docker and why is it used?**

> Docker is an open-source platform that automates the deployment of applications inside lightweight, portable containers. It solves the classic "it works on my machine" problem by bundling the application code with all its dependencies, configurations, and runtime into a single container. This ensures consistency across development, testing, and production environments.

---

**Q2. What is the difference between a Docker Image and a Docker Container?**

> A **Docker Image** is a read-only, immutable template built from a Dockerfile — like a class in OOP. A **Docker Container** is a running instance of that image — like an object instantiated from a class. You can run multiple containers from the same image, each isolated and independent.

---

**Q3. What is the difference between Docker and a Virtual Machine?**

| Feature | Docker Container | Virtual Machine |
|---|---|---|
| OS | Shares host OS kernel | Has its own OS |
| Size | MBs | GBs |
| Startup | Milliseconds | Minutes |
| Isolation | Process-level | Full hardware-level |
| Performance | Near-native | Overhead due to hypervisor |
| Use case | Microservices, apps | Full OS isolation needed |

---

**Q4. What is a Dockerfile?**

> A Dockerfile is a text script containing a series of instructions that Docker uses to build an image automatically. Each instruction creates a new layer in the image. Instructions include `FROM`, `RUN`, `COPY`, `ENV`, `EXPOSE`, `CMD`, and `ENTRYPOINT`.

---

**Q5. What is Docker Hub?**

> Docker Hub is Docker's official public registry where images are stored and shared. It hosts official images (like `nginx`, `python`, `ubuntu`) and allows users to push and pull their own images. Private registries can also be hosted using AWS ECR, GCP GCR, or self-hosted Harbor.

---

**Q6. What is the difference between `CMD` and `ENTRYPOINT`?**

> - `CMD` provides default arguments for the container but can be overridden by `docker run` arguments.
> - `ENTRYPOINT` defines the executable that always runs; it is not overridden (unless `--entrypoint` flag is used).
> - When used together: `ENTRYPOINT` is the executable and `CMD` provides default arguments to it.

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
# Runs: python app.py
# Override: docker run myimage script.py → python script.py
```

---

**Q7. What is the difference between `COPY` and `ADD`?**

> Both copy files into the image, but `ADD` has extra features: it can fetch files from a URL and auto-extract tar archives. Best practice is to always use `COPY` unless you specifically need `ADD`'s extra capabilities, as `COPY` is more explicit and predictable.

---

### Intermediate Level

---

**Q8. What are Docker Volumes and why are they important?**

> Volumes are Docker-managed persistent storage that exist outside the container's writable layer. They are important because:
> - Container data is ephemeral — deleted when container is removed
> - Volumes persist data across container restarts and re-creations
> - Volumes can be shared between multiple containers
> - They are more performant than bind mounts for write-heavy workloads

---

**Q9. How does Docker networking work? Explain bridge networking.**

> Docker creates a virtual network interface (docker0) on the host. By default, containers use the **bridge** driver, which creates an isolated internal network. Containers on the same bridge can communicate using their names as DNS hostnames. External traffic reaches containers only through explicitly published ports (`-p host_port:container_port`).

---

**Q10. What is Docker Compose and when would you use it?**

> Docker Compose is a tool for defining and running multi-container applications using a `docker-compose.yml` file. It allows you to define all services, networks, and volumes in a single file and manage them with simple commands. Used when:
> - Running apps with multiple services (web + db + cache)
> - Local development environments
> - CI/CD pipelines for integration testing

---

**Q11. What is a multi-stage build and what are its benefits?**

> Multi-stage builds allow you to use multiple `FROM` instructions in a single Dockerfile. The key benefit is producing minimal final images:
> - Stage 1 (builder): Install all build tools, compile code
> - Stage 2 (final): Copy only the compiled artifact into a clean base image
> - Result: Final image has no compilers, test tools, or source code — just the binary

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

---

**Q12. How do you reduce Docker image size?**

> 1. Use minimal base images (`alpine`, `slim`, `distroless`, or `scratch`)
> 2. Use multi-stage builds to separate build and runtime
> 3. Combine RUN commands to reduce layers
> 4. Clean up package caches in the same RUN layer (`rm -rf /var/lib/apt/lists/*`)
> 5. Use `.dockerignore` to exclude unnecessary files
> 6. Avoid installing unnecessary packages
> 7. Use `--no-cache` flag for package managers (`pip install --no-cache-dir`)

---

**Q13. What is layer caching in Docker and how do you optimize for it?**

> Docker caches each layer. If a layer hasn't changed, Docker reuses the cached version instead of rebuilding. To optimize:
> - Copy dependency files (`package.json`, `requirements.txt`) before source code
> - Install dependencies as a separate step (these change less often than code)
> - Put frequently changing instructions (like `COPY . .`) at the end

```dockerfile
# Good: dependencies cached separately from code changes
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .                   # only this layer rebuilds on code changes
```

---

**Q14. How do you pass environment variables to a Docker container?**

```bash
# 1. At runtime via -e flag
docker run -e DB_HOST=localhost myapp

# 2. From a file
docker run --env-file .env myapp

# 3. In Dockerfile (baked in, not recommended for secrets)
ENV DB_HOST=localhost

# 4. In docker-compose.yml
environment:
  - DB_HOST=localhost
env_file:
  - .env
```

---

**Q15. What is the difference between `docker stop` and `docker kill`?**

> - `docker stop`: Sends `SIGTERM` to the main process, giving it time to gracefully shut down (default 10-second grace period), then sends `SIGKILL`.
> - `docker kill`: Sends `SIGKILL` immediately, forcefully terminating the container without cleanup. Use stop for graceful shutdown; kill when a container is unresponsive.

---

### Advanced Level

---

**Q16. How does Docker achieve container isolation?**

> Docker uses two key Linux kernel features:
> - **Namespaces**: Provide isolated views of system resources — each container gets its own PID tree, network stack, hostname, filesystem mounts, IPC, and user IDs.
> - **cgroups (Control Groups)**: Limit and isolate resource usage — CPU, memory, disk I/O, and network bandwidth per container.
> Together these make containers appear as isolated systems while sharing the host kernel.

---

**Q17. What is the difference between a bind mount and a named volume?**

| Feature | Named Volume | Bind Mount |
|---|---|---|
| Managed by | Docker | Host OS |
| Portability | High (Docker manages path) | Low (host path dependent) |
| Performance | Optimized by Docker | Direct host filesystem |
| Backup | `docker volume` commands | Standard host file tools |
| Use case | Production data, databases | Development (live code reload) |

---

**Q18. How do you handle secrets in Docker?**

> - **Avoid**: Baking secrets into images or using ENV for sensitive values
> - **Docker Secrets** (Swarm): `docker secret create` — secrets mounted as files in `/run/secrets/`
> - **Environment files**: `.env` files mounted at runtime (not committed to git)
> - **Secret managers**: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault with init containers
> - **Build secrets** (BuildKit): `RUN --mount=type=secret,id=mykey cat /run/secrets/mykey`

---

**Q19. What is the Docker content trust and image signing?**

> Docker Content Trust (DCT) uses Notary to sign and verify images. When enabled (`DOCKER_CONTENT_TRUST=1`), Docker only pulls and runs signed images. This prevents tampering and ensures image integrity. Images are signed with private keys and verified using public keys stored in a Notary server.

---

**Q20. What is Docker Swarm and how does it differ from Kubernetes?**

| Feature | Docker Swarm | Kubernetes |
|---|---|---|
| Complexity | Simple, easy setup | Complex, steep learning curve |
| Scaling | Manual or limited auto-scale | Advanced HPA, VPA, cluster autoscaler |
| Ecosystem | Docker native | Huge ecosystem (CNCF) |
| Load Balancing | Built-in | Needs Ingress controller |
| Rolling Updates | Yes | Yes, more control |
| Self-healing | Basic | Advanced |
| Best For | Simple deployments | Enterprise, complex microservices |

---

**Q21. How do you debug a Docker container?**

```bash
# Access running container shell
docker exec -it container_name bash

# View real-time logs
docker logs -f container_name

# Inspect container state, env, mounts, network
docker inspect container_name

# Check resource usage
docker stats container_name

# Start stopped container with override entrypoint
docker run -it --entrypoint /bin/sh myimage

# Copy log files out
docker cp container_name:/var/log/app.log ./app.log

# Run ephemeral debug container sharing namespace
docker run -it --pid=container:my_app --net=container:my_app nicolaka/netshoot
```

---

**Q22. What is BuildKit and what are its advantages?**

> BuildKit is Docker's next-generation build engine (default since Docker 23.0). Advantages:
> - **Parallel execution**: Independent build stages run concurrently
> - **Better caching**: More granular cache control with `--mount=type=cache`
> - **Build secrets**: Mount secrets without baking them into layers
> - **SSH forwarding**: Pass SSH keys during build without storing them
> - **Smaller context**: Only sends needed files to daemon
> - **Better output**: Progress display with detailed timing

```bash
# Enable BuildKit (older Docker versions)
DOCKER_BUILDKIT=1 docker build .
```

---

**Q23. Explain the Docker container lifecycle.**

```
Created ──start──► Running ──pause──► Paused
                      │                  │
                      │◄──────unpause────┘
                      │
                   stop/kill
                      │
                      ▼
                   Exited ──start──► Running
                      │
                     rm
                      │
                      ▼
                   Deleted
```

---

**Q24. How do you implement health checks and what happens when they fail?**

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

> States: `starting` → `healthy` / `unhealthy`
> - In Docker Swarm: unhealthy containers are automatically replaced
> - In Kubernetes (via liveness probe): unhealthy containers are restarted
> - In standalone Docker: container keeps running but `docker ps` shows `(unhealthy)`

---

**Q25. What are some Docker security best practices?**

> 1. **Run as non-root user**: `USER appuser` in Dockerfile
> 2. **Use minimal base images**: Reduces attack surface (`alpine`, `distroless`)
> 3. **Scan images for vulnerabilities**: `docker scout`, Trivy, Snyk
> 4. **Never store secrets in images**: Use Docker secrets or secret managers
> 5. **Read-only filesystem**: `docker run --read-only`
> 6. **Drop Linux capabilities**: `docker run --cap-drop ALL --cap-add NET_BIND_SERVICE`
> 7. **Use signed images**: Enable Docker Content Trust
> 8. **Limit resources**: `--memory`, `--cpus` flags
> 9. **Use networks wisely**: Internal networks for backend services
> 10. **Keep images updated**: Regularly rebuild to get security patches

---

## Quick Reference Cheat Sheet

```bash
# ── Images ──────────────────────────────────────────────
docker pull <image>              # Download image
docker images                    # List images
docker rmi <image>               # Remove image
docker build -t name:tag .       # Build from Dockerfile
docker history <image>           # View layers

# ── Containers ──────────────────────────────────────────
docker run -d -p 80:80 nginx     # Run detached with port
docker run -it ubuntu bash       # Run interactive
docker ps / docker ps -a         # List containers
docker stop / start / restart    # Lifecycle
docker rm <container>            # Remove container
docker exec -it <name> bash      # Shell into container
docker logs -f <name>            # Stream logs
docker inspect <name>            # Full details
docker stats                     # Resource usage

# ── Volumes ──────────────────────────────────────────────
docker volume create vol         # Create volume
docker volume ls / inspect       # List/inspect
docker run -v vol:/data nginx     # Mount volume

# ── Networks ─────────────────────────────────────────────
docker network create mynet      # Create network
docker network ls / inspect      # List/inspect
docker network connect mynet c1  # Connect container

# ── Compose ─────────────────────────────────────────────
docker compose up -d             # Start all services
docker compose down              # Stop all services
docker compose logs -f           # Stream logs
docker compose exec web bash     # Shell into service

# ── Cleanup ─────────────────────────────────────────────
docker system prune -a           # Remove all unused resources
docker image prune -a            # Remove unused images
docker container prune           # Remove stopped containers
docker volume prune              # Remove unused volumes
```

---

*Last updated: 2025 | Docker Engine 26.x*

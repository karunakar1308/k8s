 Docker & Kubernetes Documentation

---

# PART 1: DOCKER FUNDAMENTALS & ADVANCED

## 1. Introduction to Containerization

### What is Containerization?

**Containerization** is a lightweight form of virtualization that packages an application and its dependencies together in an isolated environment called a **container**. Unlike traditional virtual machines (VMs), containers share the host OS kernel, making them more efficient and faster to start.

### Key Differences: Containers vs Virtual Machines

| Aspect | Virtual Machines | Containers |
|--------|------------------|------------|
| **Size** | GBs (includes full OS) | MBs (shares host kernel) |
| **Startup Time** | Minutes | Seconds |
| **Resource Usage** | Heavy (hypervisor overhead) | Lightweight |
| **Isolation** | Complete OS-level isolation | Process-level isolation |
| **Portability** | Less portable | Highly portable |
| **Performance** | Slower | Near-native performance |

### Benefits of Containerization

- ✅ **Consistency** across environments (dev, test, prod)
- ✅ **Faster deployment** and scaling
- ✅ **Efficient resource** utilization
- ✅ **Microservices architecture** enablement
- ✅ **Version control** for infrastructure
- ✅ **Simplified dependency** management

> **📌 Learn more:**
> - [Docker Overview](https://docs.docker.com/get-started/overview/)
> - [Containers vs VMs](https://www.docker.com/resources/what-container/)

### Interview Scenario

**Q: "Explain the difference between containers and virtual machines."**

**A:** "Containers virtualize the operating system, sharing the host kernel, while VMs virtualize hardware, each running a full OS. This makes containers lighter (MBs vs GBs), faster to start (seconds vs minutes), and more resource-efficient. However, VMs provide stronger isolation. In my experience, I've used containers for microservices deployments where we needed fast scaling and VMs for workloads requiring complete OS isolation."

---

## 2. Docker Architecture & Components

### Docker Architecture Overview

Docker uses a **client-server architecture**:

```
┌─────────────┐          ┌──────────────┐         ┌─────────────┐
│   Docker    │  REST    │    Docker    │         │   Docker    │
│   Client    │ ────────>│   Daemon     │────────>│  Registry   │
│  (docker)   │   API    │  (dockerd)   │         │ (Docker Hub)│
└─────────────┘          └──────────────┘         └─────────────┘
```

### Key Components

#### 1. **Docker Client (`docker` CLI)**
- Command-line interface for users
- Sends commands to Docker daemon via REST API
- Can communicate with remote daemons
- **Common commands:**
  ```bash
  docker build    # Build an image
  docker run      # Run a container
  docker push     # Push to registry
  docker pull     # Pull from registry
  ```

#### 2. **Docker Daemon (`dockerd`)**
- Background service running on the host
- Manages Docker objects: images, containers, networks, volumes
- Listens to Docker API requests
- Can communicate with other daemons for distributed systems

#### 3. **Docker Registry**
- Stores Docker images
- **Public registries:**
  - Docker Hub (default)
- **Private registries:**
  - AWS ECR, Google GCR, Azure ACR, Harbor, Artifactory
- **Commands:**
  ```bash
  docker pull nginx:latest
  docker push myrepo/myimage:v1.0
  ```

### Docker Objects

#### **Images**
- Read-only templates with instructions for creating containers
- Built from Dockerfile
- Layered filesystem (each instruction creates a layer)
- Cached layers for efficiency
- **Base images:** ubuntu, alpine, nginx, etc.

**Example Dockerfile:**
```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Start command
CMD ["npm", "start"]
```

#### **Containers**
- Runnable instances of images
- Isolated from other containers and host
- Can be started, stopped, moved, deleted
- **Ephemeral by default** (data lost when removed)
- Defined by image and configuration options

**Lifecycle commands:**
```bash
# Create and start container
docker run -d --name myapp -p 8080:80 nginx

# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop myapp

# Start stopped container
docker start myapp

# Remove container
docker rm myapp

# View container logs
docker logs myapp

# Execute command in container
docker exec -it myapp /bin/bash
```

---

## 3. Docker Images & Dockerfile

### Understanding Docker Images

Docker images are built from a series of **layers**, each representing a Dockerfile instruction. Layers are **cached** and **reusable**, making builds faster and more efficient.

```
┌──────────────────────┐
│  CMD ["npm","start"] │  Layer 5
├──────────────────────┤
│  COPY . .            │  Layer 4
├──────────────────────┤
│  RUN npm install     │  Layer 3
├──────────────────────┤
│  COPY package.json   │  Layer 2
├──────────────────────┤
│  FROM node:18        │  Layer 1 (Base)
└──────────────────────┘
```

### Dockerfile Best Practices

#### 1. **Use Specific Base Image Tags**

❌ **Bad:**
```dockerfile
FROM node
```

✅ **Good:**
```dockerfile
FROM node:18-alpine
```

#### 2. **Minimize Layers**

❌ **Bad:**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
```

✅ **Good:**
```dockerfile
RUN apt-get update && apt-get install -y \\
    curl \\
    vim \\
    && rm -rf /var/lib/apt/lists/*
```

#### 3. **Leverage Build Cache**

✅ **Good:** Copy dependency files first, then install
```dockerfile
# Dependencies change less frequently
COPY package*.json ./
RUN npm install

# Source code changes more frequently
COPY . .
```

#### 4. **Use .dockerignore**

Create a `.dockerignore` file:
```
node_modules
.git
.env
*.md
Dockerfile
.dockerignore
```

### Multi-Stage Builds

**Multi-stage builds** reduce final image size by separating build and runtime environments.

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm install --production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**Benefits:**
- Smaller final image (no build tools)
- Faster deployment
- Better security (fewer attack vectors)

---

## 4. Docker Networking

### Network Drivers

Docker provides several network drivers:

| Driver | Description | Use Case |
|--------|-------------|----------|
| **bridge** | Default network driver | Standalone containers on single host |
| **host** | Remove network isolation | High performance, no port mapping |
| **overlay** | Multi-host networking | Swarm services, multi-host apps |
| **macvlan** | Assign MAC address | Legacy apps needing physical network |
| **none** | Disable networking | Maximum isolation |

### Basic Network Commands

```bash
# List networks
docker network ls

# Create custom bridge network
docker network create mynet

# Run container on custom network
docker run -d --name app1 --network mynet nginx

# Connect running container to network
docker network connect mynet app2

# Disconnect from network
docker network disconnect mynet app2

# Inspect network
docker network inspect mynet
```

### Container Communication

**Containers on the same network can communicate by name:**

```bash
# Create network
docker network create appnet

# Run database
docker run -d --name db --network appnet postgres

# Run app (can connect to 'db' by name)
docker run -d --name webapp --network appnet -e DB_HOST=db myapp
```

**Port Mapping:**
```bash
# Map container port 80 to host port 8080
docker run -d -p 8080:80 nginx

# Map all exposed ports to random host ports
docker run -d -P nginx

# Bind to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx
```

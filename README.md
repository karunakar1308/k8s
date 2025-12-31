Docker, Kubernetes & ArgoCD: Complete Guide from Basics to Advanced
With Interview Scenarios and Real-World Examples

TABLE OF CONTENTS

PART 1: DOCKER FUNDAMENTALS & ADVANCED
1. Introduction to Containerization
2. Docker Architecture & Components
3. Docker Images & Dockerfile
4. Docker Containers Lifecycle
5. Docker Networking
6. Docker Volumes & Storage
7. Docker Compose
8. Multi-stage Builds & Optimization
9. Docker Security Best Practices
10. Docker Registry & Image Management

PART 2: KUBERNETES FUNDAMENTALS
11. What is Kubernetes?
12. Kubernetes Architecture
13. Control Plane Components
14. Node Components
15. Kubernetes Objects & API

PART 3: KUBERNETES WORKLOADS
16. Pods
17. ReplicaSets
18. Deployments
19. StatefulSets
20. DaemonSets
21. Jobs & CronJobs

PART 4: KUBERNETES NETWORKING
22. Services (ClusterIP, NodePort, LoadBalancer)
23. Ingress & Ingress Controllers
24. Network Policies
25. CNI Plugins (Calico, Cilium, Flannel)
26. DNS in Kubernetes
27. Service Mesh Fundamentals

PART 5: KUBERNETES CONFIGURATION & STORAGE
28. ConfigMaps
29. Secrets
30. Persistent Volumes (PV)
31. Persistent Volume Claims (PVC)
32. Storage Classes
33. StatefulSet Storage

PART 6: KUBERNETES SECURITY
34. RBAC (Role-Based Access Control)
35. Service Accounts
36. Pod Security Standards
37. Network Policies for Security
38. Admission Controllers
39. Security Contexts
40. Secrets Management (External Secrets, Vault)

PART 7: KUBERNETES SCALING & AUTOSCALING
41. Horizontal Pod Autoscaler (HPA)
42. Vertical Pod Autoscaler (VPA)
43. Cluster Autoscaler
44. KEDA (Kubernetes Event-Driven Autoscaling)

PART 8: KUBERNETES OBSERVABILITY
45. Logging (Fluentd, Fluent Bit, EFK Stack)
46. Monitoring (Prometheus, Grafana)
47. Metrics Server
48. Distributed Tracing (Jaeger, Zipkin)
49. Debugging Pods & Troubleshooting

PART 9: SERVICE MESH ADVANCED
50. Istio Architecture & Components
51. Linkerd Overview
52. Traffic Management
53. mTLS & Security in Service Mesh
54. Observability with Service Mesh
55. Canary Deployments with Istio

PART 10: ADVANCED DEPLOYMENT STRATEGIES
56. Rolling Updates
57. Blue-Green Deployments
58. Canary Deployments
59. A/B Testing
60. GitOps Principles

PART 11: ARGOCD BASICS TO ADVANCED
61. Introduction to GitOps
62. ArgoCD Architecture
63. Installing ArgoCD
64. Creating Applications in ArgoCD
65. App of Apps Pattern
66. Helm Integration with ArgoCD
67. Kustomize Integration
68. ApplicationSets
69. Sync Policies & Strategies
70. Progressive Delivery with ArgoCD Rollouts
71. Multi-cluster Management
72. ArgoCD Security & RBAC
73. ArgoCD Notifications & Webhooks
74. ArgoCD Best Practices

PART 12: PRODUCTION BEST PRACTICES
75. High Availability Clusters
76. Disaster Recovery & Backup (Velero)
77. Multi-cluster Strategies
78. Cost Optimization
79. Performance Tuning
80. Interview Preparation Tips

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART 1: DOCKER FUNDAMENTALS & ADVANCED

1. INTRODUCTION TO CONTAINERIZATION

What is Containerization?
Containerization is a lightweight form of virtualization that packages an application and its dependencies together in an isolated environment called a container. Unlike traditional virtual machines (VMs), containers share the host OS kernel, making them more efficient and faster to start.

Key Differences: Containers vs Virtual Machines

| Aspect | Virtual Machines | Containers |
| Size | GBs (includes full OS) | MBs (shares host kernel) |
| Startup Time | Minutes | Seconds |
| Resource Usage | Heavy (hypervisor overhead) | Lightweight |
| Isolation | Complete OS-level isolation | Process-level isolation |
| Portability | Less portable | Highly portable |
| Performance | Slower | Near-native performance |

Benefits of Containerization:
• Consistency across environments (dev, test, prod)
• Faster deployment and scaling
• Efficient resource utilization
• Microservices architecture enablement
• Version control for infrastructure
• Simplified dependency management

Learn more:
• Docker Overview: https://docs.docker.com/get-started/overview/
• Containers vs VMs: https://www.docker.com/resources/what-container/

Interview Scenario:
Q: "Explain the difference between containers and virtual machines."
A: "Containers virtualize the operating system, sharing the host kernel, while VMs virtualize hardware, each running a full OS. This makes containers lighter (MBs vs GBs), faster to start (seconds vs minutes), and more resource-efficient. However, VMs provide stronger isolation. In my experience, I've used containers for microservices deployments where we needed fast scaling and VMs for workloads requiring complete OS isolation."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

2. DOCKER ARCHITECTURE & COMPONENTS

Docker Architecture Overview
Docker uses a client-server architecture with three main components:

1. Docker Client (docker CLI)
• Command-line interface for users
• Sends commands to Docker daemon via REST API
• Can communicate with remote daemons
• Commands: docker build, docker run, docker push, etc.

2. Docker Daemon (dockerd)
• Background service running on the host
• Manages Docker objects: images, containers, networks, volumes
• Listens to Docker API requests
• Can communicate with other daemons for distributed systems

3. Docker Registry
• Stores Docker images
• Docker Hub: public registry (default)
• Private registries: AWS ECR, Google GCR, Azure ACR, Harbor, Artifactory
• Commands: docker pull, docker push

Docker Objects:

Images:
• Read-only templates with instructions for creating containers
• Built from Dockerfile
• Layered filesystem (each instruction creates a layer)
• Cached layers for efficiency
• Base images from Docker Hub (ubuntu, alpine, nginx, etc.)

Containers:
• Runnable instances of images
• Isolated from other containers and host
• Can be started, stopped, moved, deleted
• Defined by image and configuration options
• Ephemeral by default (data lost when removed)

Networks:
• Enable communication between containers
• Types: bridge, host, overlay, macvlan, none
• Network drivers for different use cases

Volumes:
• Persistent data storage
• Managed by Docker
• Survive container deletion
• Can be shared between containers

Docker Engine Components:

1. containerd
• Industry-standard container runtime
• Manages container lifecycle
• Image transfer and storage
• Supervises container execution

2. runc
• Low-level container runtime
• OCI (Open Container Initiative) compliant
• Creates and runs containers

3. shim
• Keeps STDIN/STDOUT open
• Reports exit status
• Allows daemon restart without killing containers

Learn more:
• Docker Architecture: https://docs.docker.com/get-started/overview/#docker-architecture
• containerd: https://containerd.io/
• OCI Runtime Spec: https://github.com/opencontainers/runtime-spec

Interview Scenario:
Q: "Walk me through what happens when you run 'docker run nginx'?"
A: "When I execute 'docker run nginx':
1. Docker client sends the command to the Docker daemon via REST API
2. Daemon checks if the 'nginx' image exists locally
3. If not, it pulls from Docker Hub (default registry)
4. Daemon creates a new container from the image
5. Allocates a read-write filesystem layer on top of image layers
6. Creates a network interface and assigns an IP (default bridge network)
7. Starts the container and executes the specified command (in nginx's case, starts the web server)
8. containerd and runc handle the actual container creation and execution

In production, I've automated this with specific tags, custom networks, volume mounts, and environment variables based on the application requirements."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

3. DOCKER IMAGES & DOCKERFILE

Docker Images
A Docker image is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, libraries, and settings.

Image Layers:
• Images are built in layers (each Dockerfile instruction creates a layer)
• Layers are cached and reused
• Read-only layers from image + writable layer for container
• Sharing layers between images saves disk space
• Layer changes are tracked using copy-on-write (COW)

Image Naming Convention:
[registry]/[namespace]/[repository]:[tag]
Examples:
• docker.io/library/nginx:latest
• myregistry.azurecr.io/myapp:v1.2.3
• 123456789.dkr.ecr.us-east-1.amazonaws.com/webapp:prod

Common Docker Image Commands:

docker pull <image>           # Download image from registry
docker images                 # List local images
docker rmi <image>            # Remove image
docker tag <source> <target>  # Tag an image
docker push <image>           # Upload image to registry
docker build -t <name> .      # Build image from Dockerfile
docker history <image>        # Show image layers
docker inspect <image>        # Detailed image information
docker save -o file.tar <img> # Export image to tar
docker load -i file.tar       # Import image from tar

Dockerfile
A Dockerfile is a text file containing instructions to build a Docker image.

Dockerfile Instructions:

FROM - Base image
  FROM ubuntu:20.04
  FROM python:3.9-slim AS builder
  • Must be first instruction (except ARG before FROM)
  • Multi-stage builds use multiple FROM

RUN - Execute commands during build
  RUN apt-get update && apt-get install -y nginx
  RUN pip install -r requirements.txt
  • Each RUN creates a new layer
  • Combine commands with && to reduce layers

COPY - Copy files from host to image
  COPY app.py /app/
  COPY requirements.txt .
  • Preferred over ADD for simple copy

ADD - Copy with additional features
  ADD archive.tar.gz /app/
  ADD https://example.com/file.txt /tmp/
  • Auto-extracts tar files
  • Can download from URLs

WORKDIR - Set working directory
  WORKDIR /app
  • Creates directory if doesn't exist
  • Affects subsequent instructions

ENV - Set environment variables
  ENV NODE_ENV=production
  ENV PATH=/app/bin:$PATH
  • Available during build and runtime

ARG - Build-time variables
  ARG VERSION=1.0
  ARG BUILD_DATE
  • Only available during build
  • Can be overridden: docker build --build-arg VERSION=2.0

EXPOSE - Document which ports to expose
  EXPOSE 80
  EXPOSE 443
  • Informational only (doesn't publish ports)
  • Use -p flag in docker run to actually publish

CMD - Default command when container starts
  CMD ["python", "app.py"]
  CMD ["nginx", "-g", "daemon off;"]
  • Can be overridden by docker run arguments
  • Only one CMD per Dockerfile (last one wins)

ENTRYPOINT - Configure container as executable
  ENTRYPOINT ["python", "app.py"]
  • Not easily overridden (use --entrypoint)
  • CMD becomes arguments to ENTRYPOINT

VOLUME - Create mount point
  VOLUME /data
  • Better to define at runtime

USER - Set user for subsequent commands
  USER appuser
  • Security best practice (don't run as root)

HEALTHCHECK - Container health check
  HEALTHCHECK CMD curl -f http://localhost/ || exit 1

LABEL - Add metadata
  LABEL version="1.0" maintainer="user@example.com"

Example Dockerfile (Python Flask App):

FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

ENV FLASK_APP=app.py

CMD ["flask", "run", "--host=0.0.0.0"]

Example Dockerfile (Node.js App with Multi-stage):

# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]

Building Images:

docker build -t myapp:1.0 .                    # Build from current directory
docker build -t myapp:1.0 -f Dockerfile.prod . # Use specific Dockerfile
docker build --no-cache -t myapp:1.0 .         # Build without cache
docker build --build-arg VERSION=2.0 -t myapp:2.0 .

.dockerignore File:
Like .gitignore, excludes files from build context

node_modules
*.log
.git
.env
dist

Best Practices:
1. Use specific base image tags (not 'latest')
2. Minimize layers (combine RUN commands)
3. Order instructions from least to most frequently changing
4. Use .dockerignore to exclude unnecessary files
5. Don't install unnecessary packages
6. Use multi-stage builds to reduce final image size
7. Run as non-root user
8. Scan images for vulnerabilities

Learn more:
• Dockerfile Reference: https://docs.docker.com/engine/reference/builder/
• Best Practices: https://docs.docker.com/develop/dev-best-practices/
• Multi-stage Builds: https://docs.docker.com/build/building/multi-stage/

Interview Scenario:
Q: "How would you optimize a Dockerfile for a Node.js application?"
A: "Several optimization strategies:

1. Multi-stage builds - Use one stage for building (node:16) and another slim stage for production (node:16-alpine) to reduce final image size

2. Layer caching - Copy package.json first, run npm install, then copy source code. This way, dependencies are cached unless package.json changes

3. Minimize layers - Combine RUN commands with && to reduce image layers

4. Use .dockerignore - Exclude node_modules, .git, tests from build context

5. Security - Run as non-root USER node, scan for vulnerabilities with tools like Trivy

6. Specific base images - Use exact versions like node:16.14-alpine instead of 'latest'

In production, I've reduced Node.js images from 1GB to 150MB using these techniques, significantly improving deployment speed and security posture."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

4. DOCKER CONTAINER LIFECYCLE

Container States:
1. Created - Container has been created but not started
2. Running - Container is currently executing
3. Paused - Container processes are paused
4. Stopped - Container has exited
5. Deleted - Container has been removed

Container Lifecycle Commands:

Creating & Starting:
docker create <image>              # Create container (doesn't start)
docker start <container>           # Start stopped container
docker run <image>                 # Create and start in one command
docker run -d <image>              # Run in detached mode (background)
docker run -it <image> /bin/bash   # Interactive terminal
docker run --name myapp <image>    # Assign name to container
docker run --rm <image>            # Auto-remove when stopped

Common docker run Options:
-d, --detach          Run in background
-it                   Interactive with TTY
--name                Assign container name
-p, --publish         Publish ports (host:container)
-v, --volume          Mount volumes
-e, --env             Set environment variables
--network             Connect to network
--restart             Restart policy (no, on-failure, always, unless-stopped)
--memory              Memory limit
--cpus                CPU limit
--rm                  Auto-remove when stopped
--privileged          Give extended privileges
--user                Set user
--workdir             Set working directory
--entrypoint          Override entrypoint

Examples:
docker run -d -p 8080:80 --name web nginx
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secret -v /data:/var/lib/mysql mysql
docker run -it --rm ubuntu:20.04 /bin/bash
docker run -d --restart=unless-stopped --memory=512m --cpus=0.5 myapp

Managing Running Containers:
docker ps                   # List running containers
docker ps -a                # List all containers
docker stop <container>     # Graceful stop (SIGTERM)
docker kill <container>     # Force stop (SIGKILL)
docker restart <container>  # Restart container
docker pause <container>    # Pause processes
docker unpause <container>  # Resume processes
docker rm <container>       # Remove stopped container
docker rm -f <container>    # Force remove running container

Inspecting & Interacting:
docker logs <container>         # View logs
docker logs -f <container>      # Follow logs (tail -f)
docker logs --tail 100 <cont>   # Last 100 lines
docker exec -it <cont> bash     # Execute command in running container
docker attach <container>       # Attach to running container
docker top <container>          # Display processes
docker stats                    # Live resource usage
docker inspect <container>      # Detailed information
docker port <container>         # Port mappings
docker diff <container>         # Filesystem changes

Copying Files:
docker cp <container>:/path/file ./local  # Copy from container to host
docker cp ./local <container>:/path       # Copy from host to container

Exporting & Importing:
docker export <container> > backup.tar    # Export container filesystem
docker import backup.tar myimage:tag      # Import as image

Committing Changes:
docker commit <container> newimage:tag    # Create image from container
# Note: Not recommended for production (use Dockerfile instead)

Container Resource Management:

Memory Limits:
docker run --memory=512m myapp              # Hard limit
docker run --memory=512m --memory-reservation=256m myapp  # Soft limit

CPU Limits:
docker run --cpus=1.5 myapp                 # 1.5 CPUs
docker run --cpu-shares=512 myapp           # Relative weight

Restart Policies:
no                    Never restart (default)
on-failure[:max]      Restart on non-zero exit
always                Always restart (even manual stop)
unless-stopped        Always restart unless manually stopped

Example:
docker run -d --restart=on-failure:3 myapp

Health Checks:

In Dockerfile:
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

At runtime:
docker run --health-cmd="curl -f http://localhost/health" \
           --health-interval=30s myapp

Check health:
docker inspect --format='{{.State.Health.Status}}' <container>

Learn more:
• docker run Reference: https://docs.docker.com/engine/reference/run/
• Container Lifecycle: https://docs.docker.com/engine/reference/commandline/container/
• Resource Constraints: https://docs.docker.com/config/containers/resource_constraints/

Interview Scenario:
Q: "A container keeps restarting in production. How would you troubleshoot?"
A: "My troubleshooting approach:

1. Check logs: docker logs <container> to see error messages

2. Inspect restart count: docker inspect <container> | grep RestartCount

3. Check resource limits: docker stats to see if hitting memory/CPU limits

4. Review health checks: docker inspect --format='{{.State.Health}}' 

5. Examine application logs inside: docker exec -it <container> cat /var/log/app.log

6. Check dependencies: Ensure databases, APIs are reachable (network issues)

7. Review recent changes: Git history, recent deployments, configuration changes

8. Test locally: docker run with same config to reproduce

Real example: I once debugged a restart loop caused by OOMKilled (out of memory). Docker stats showed memory spiking to limit. Increased memory limit and added memory leak detection in the application."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

5. DOCKER NETWORKING

Docker provides several networking options to enable container communication.

Network Drivers:

1. BRIDGE (default)
• Private internal network on the host
• Containers on same bridge can communicate
• Port mapping required for external access
• Default bridge: docker0 (172.17.0.0/16)
• Custom bridges: Better isolation, automatic DNS

Learn more about Bridge Networking: https://docs.docker.com/network/bridge/

2. HOST
• Removes network isolation
• Container uses host's network stack directly
• No port mapping needed
• Better performance but less secure
• Port conflicts possible

Learn more about Host Networking: https://docs.docker.com/network/host/

3. OVERLAY
• Multi-host networking for Docker Swarm
• Enables container communication across multiple Docker hosts
• Uses VXLAN encapsulation
• Required for Swarm services

Learn more about Overlay Networks: https://docs.docker.com/network/overlay/

4. MACVLAN
• Assigns MAC address to container
• Container appears as physical device on network
• Used for legacy applications requiring direct network access

Learn more about Macvlan: https://docs.docker.com/network/macvlan/

5. NONE
• Disables all networking
• Complete network isolation
• Loopback interface only

Network Commands:

docker network ls                          # List networks
docker network create mynet                # Create bridge network
docker network create --driver overlay my-overlay  # Overlay network
docker network inspect mynet               # Detailed info
docker network connect mynet container1    # Connect running container
docker network disconnect mynet container1 # Disconnect
docker network rm mynet                    # Remove network
docker network prune                       # Remove unused networks

Custom Bridge Network:

docker network create --driver bridge \
  --subnet=172.18.0.0/16 \
  --gateway=172.18.0.1 \
  --opt "com.docker.network.bridge.name"="my-bridge" \
  my-custom-net

Using Networks:

# Run container on specific network
docker run -d --name web --network my-custom-net nginx

# Connect to multiple networks
docker network connect frontend web
docker network connect backend web

Port Mapping:

-p HOST_PORT:CONTAINER_PORT

Examples:
docker run -d -p 8080:80 nginx              # Map 8080 to 80
docker run -d -p 127.0.0.1:8080:80 nginx    # Bind to localhost only
docker run -d -p 3000-3005:3000-3005 app    # Range mapping
docker run -d -P nginx                      # Random port mapping

Container DNS:
• Custom bridge networks have built-in DNS
• Containers can resolve each other by name
• Automatic service discovery

Example:
docker network create myapp-net
docker run -d --name db --network myapp-net postgres
docker run -d --name web --network myapp-net -e DB_HOST=db myapp
# 'web' can connect to 'db' using hostname 'db'

Links (Legacy - not recommended):
docker run --name web --link db:database nginx
# Use custom networks instead

Network Troubleshooting:

# Inspect container's network settings
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container

# Test connectivity
docker exec container1 ping container2
docker exec container curl http://container2:8080

# View network namespace
docker exec container ip addr
docker exec container netstat -tulpn

Advanced: Network Plugins (CNI)
For Kubernetes and advanced scenarios:
• Calico: https://www.tigera.io/project-calico/
• Weave: https://www.weave.works/docs/net/latest/overview/
• Flannel: https://github.com/flannel-io/flannel

Learn more:
• Docker Networking Overview: https://docs.docker.com/network/
• Network Drivers: https://docs.docker.com/network/drivers/
• Docker Networking Deep Dive: https://medium.com/@sanchit0496/docker-networking-deep-dive-4312e0a2b9c4

Interview Scenario:
Q: "How would you allow two containers to communicate without exposing ports externally?"
A: "I would create a custom bridge network and attach both containers to it:

docker network create --driver bridge backend-net
docker run -d --name database --network backend-net postgres
docker run -d --name api --network backend-net -e DB_HOST=database myapi

With custom bridge networks, Docker provides built-in DNS resolution, so the 'api' container can reach the 'database' container using its container name as the hostname. No port mapping (-p) is needed since they're on the same internal network. This approach is more secure than using the default bridge or host networking, and provides better isolation. In production microservices, I've used this pattern extensively with separate networks for frontend, backend, and data tiers."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

6. DOCKER VOLUMES & STORAGE

Containers are ephemeral - data is lost when container is removed. Volumes provide persistent storage.

Storage Options:

1. Volumes (Recommended)
• Managed by Docker
• Stored in /var/lib/docker/volumes/ (Linux)
• Best for production
• Can be backed up easily
• Work on all platforms
• Can use volume drivers for remote storage

Learn more: https://docs.docker.com/storage/volumes/

2. Bind Mounts
• Mount host directory/file into container
• Full path required
• Good for development (live code reload)
• Less portable
• Host-dependent

Learn more: https://docs.docker.com/storage/bind-mounts/

3. tmpfs Mounts (Linux only)
• Stored in host memory only
• Never written to disk
• Fast, temporary storage
• Lost when container stops
• Good for secrets or temporary data

Learn more: https://docs.docker.com/storage/tmpfs/

Volume Commands:

docker volume create myvolume           # Create volume
docker volume ls                        # List volumes
docker volume inspect myvolume          # Detailed info
docker volume rm myvolume               # Remove volume
docker volume prune                     # Remove unused volumes

Using Volumes:

# Named volume
docker run -d -v myvolume:/data nginx

# Anonymous volume (Docker generates name)
docker run -d -v /data nginx

# Bind mount (absolute path)
docker run -d -v /host/path:/container/path nginx
docker run -d -v $(pwd):/app node:16

# Read-only mount
docker run -d -v myvolume:/data:ro nginx

# tmpfs mount
docker run -d --tmpfs /tmp:rw,size=100m,mode=1777 myapp

Practical Examples:

1. MySQL with Persistent Storage:
docker volume create mysql-data
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

2. Development with Live Reload:
docker run -d --name webapp \
  -v $(pwd)/src:/app/src \
  -v node_modules:/app/node_modules \
  -p 3000:3000 \
  node:16

3. Multiple Volumes:
docker run -d --name app \
  -v app-data:/data \
  -v app-logs:/var/log/app \
  -v app-config:/etc/app:ro \
  myapp

Sharing Volumes Between Containers:

# Create volume
docker volume create shared-data

# Container 1 writes
docker run -d --name writer -v shared-data:/data alpine sh -c "while true; do date > /data/timestamp.txt; sleep 5; done"

# Container 2 reads
docker run -d --name reader -v shared-data:/data alpine sh -c "while true; do cat /data/timestamp.txt; sleep 5; done"

Volumes from Another Container:

docker run -d --name data-container -v /data alpine
docker run -d --name app --volumes-from data-container myapp

Backup & Restore:

Backup:
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz /data

Restore:
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine sh -c "cd /data && tar xzf /backup/backup.tar.gz --strip 1"

Volume Drivers:
Extend Docker with storage plugins:

• Local driver (default)
• NFS driver
• AWS EBS: https://github.com/rexray/rexray
• Azure File Storage
• GCE Persistent Disk
• Ceph, GlusterFS

Example with NFS:
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  nfs-volume

Storage Best Practices:
1. Use named volumes for production data
2. Use bind mounts for development only
3. Never store data in container's writable layer
4. Regular backups of volumes
5. Use volume drivers for cloud storage in production
6. Set appropriate permissions
7. Monitor volume usage

Learn more:
• Docker Storage Overview: https://docs.docker.com/storage/
• Volume Plugins: https://docs.docker.com/engine/extend/plugins_volume/
• Storage Best Practices: https://docs.docker.com/develop/dev-best-practices/

Interview Scenario:
Q: "What's the difference between volumes and bind mounts? When would you use each?"
A: "Key differences:

Volumes:
- Managed entirely by Docker
- Stored in Docker's storage directory
- Can use volume drivers for remote storage
- Better for production
- Platform-independent
- Can't accidentally delete from host

Bind Mounts:
- Map any host directory into container  
- Require absolute paths
- Tied to host filesystem structure
- Better for development
- Allow live code changes without rebuilding

In my projects:

Development - Bind mounts:
docker run -v $(pwd)/src:/app/src node:16
(Changes to code are immediately reflected)

Production - Named volumes:
docker run -v postgres-data:/var/lib/postgresql/data postgres
(Docker manages storage, works with orchestration, can use cloud storage drivers)

I've also used bind mounts to inject configuration files that are different per environment, and volumes for all application data that needs to persist."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

7. DOCKER COMPOSE

Docker Compose is a tool for defining and running multi-container Docker applications using YAML files.

Key Concepts:
• Define multi-container apps in docker-compose.yml
• Single command to start entire application stack
• Service definition with dependencies
• Network and volume management
• Environment variable management
• Perfect for local development environments

Official Documentation: https://docs.docker.com/compose/
Compose File Reference: https://docs.docker.com/compose/compose-file/

Basic docker-compose.yml Structure:

version: '3.8'  # Compose file version

services:        # Define containers
  web:
    image: nginx:latest
    ports:
      - "80:80"
    
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret

Docker Compose Commands:

docker-compose up                  # Start all services
docker-compose up -d               # Start in detached mode
docker-compose up --build          # Rebuild images before starting
docker-compose down                # Stop and remove containers
docker-compose down -v             # Also remove volumes
docker-compose ps                  # List containers
docker-compose logs                # View logs
docker-compose logs -f web         # Follow logs for specific service
docker-compose exec web bash       # Execute command in running container
docker-compose build               # Build or rebuild services
docker-compose pull                # Pull service images
docker-compose restart             # Restart services
docker-compose stop                # Stop services (keep containers)
docker-compose start               # Start stopped services
docker-compose pause               # Pause services
docker-compose unpause             # Unpause services
docker-compose top                 # Display running processes

Complete docker-compose.yml Example (MERN Stack):

version: '3.8'

services:
  # MongoDB Database
  mongodb:
    image: mongo:5.0
    container_name: mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_PASSWORD: password123
    volumes:
      - mongodb_data:/data/db
    networks:
      - app-network
    
  # Express.js API Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: express-api
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://admin:password123@mongodb:27017/myapp?authSource=admin
    depends_on:
      - mongodb
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - app-network
    command: npm run dev
    
  # React Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: react-app
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    depends_on:
      - backend
    volumes:
      - ./frontend:/app
      - /app/node_modules
    networks:
      - app-network
    stdin_open: true
    tty: true

volumes:
  mongodb_data:
    driver: local

networks:
  app-network:
    driver: bridge

Compose File Key Sections:

1. services:
Defines containers to run

service_name:
  image: "image:tag"              # Use existing image
  build:                          # Or build from Dockerfile
    context: ./path
    dockerfile: Dockerfile.dev
    args:
      BUILD_ENV: development
  container_name: my-container    # Optional custom name
  restart: unless-stopped         # Restart policy
  ports:                          # Port mappings
    - "8080:80"
    - "443:443"
  expose:                         # Expose ports (internal only)
    - "3000"
  environment:                    # Environment variables
    - DEBUG=true
    - API_KEY=abc123
  env_file:                       # Load from file
    - .env
    - .env.local
  volumes:                        # Volume/bind mounts
    - ./data:/var/lib/data
    - named_volume:/app/uploads
  networks:                       # Connect to networks
    - frontend
    - backend
  depends_on:                     # Start order
    - database
    - redis
  command: npm run dev            # Override default command
  entrypoint: ["/bin/sh", "-c"]  # Override entrypoint
  working_dir: /app               # Working directory
  user: "1000:1000"               # Run as user
  healthcheck:                    # Health check
    test: ["CMD", "curl", "-f", "http://localhost/health"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 40s
  labels:                         # Metadata
    com.example.description: "My service"
  logging:                        # Logging configuration
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

2. volumes:
Define named volumes

volumes:
  db_data:                        # Simple volume
  app_data:
    driver: local                 # Explicit driver
    driver_opts:
      type: nfs
      o: addr=192.168.1.1,rw
      device: ":/path/to/dir"

3. networks:
Define custom networks

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

Advanced Examples:

1. WordPress with MySQL:

version: '3.8'

services:
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html

volumes:
  db_data:
  wordpress_data:

2. Microservices with Nginx Load Balancer:

version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api1
      - api2
    networks:
      - webnet
      
  api1:
    build: ./api
    environment:
      - INSTANCE=1
    networks:
      - webnet
      
  api2:
    build: ./api
    environment:
      - INSTANCE=2
    networks:
      - webnet

networks:
  webnet:

Environment Variables:

In docker-compose.yml:
environment:
  - VAR1=value1
  - VAR2=value2

From .env file:
env_file:
  - .env

.env file:
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=myapp

Variable substitution:
ports:
  - "${EXTERNAL_PORT}:80"

Multiple Compose Files:
Override configurations for different environments

Base: docker-compose.yml
Development: docker-compose.dev.yml
Production: docker-compose.prod.yml

# Use specific file
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

Scaling Services:

# Scale a service to multiple instances
docker-compose up -d --scale api=3

# Note: Can't scale if ports are explicitly mapped
# Use port ranges or no port mapping for scalable services

Docker Compose vs Kubernetes:

| Feature | Docker Compose | Kubernetes |
| Scope | Single host | Multi-host cluster |
| Orchestration | Basic | Advanced |
| Scaling | Manual | Auto-scaling |
| Self-healing | No | Yes |
| Load Balancing | Manual (nginx) | Built-in |
| Rolling Updates | No | Yes |
| Use Case | Development, small apps | Production, large-scale |

Learn more:
• Compose Best Practices: https://docs.docker.com/compose/production/
• Awesome Compose: https://github.com/docker/awesome-compose
• Compose Specification: https://compose-spec.io/

Interview Scenario:
Q: "How would you structure a development environment for a microservices application?"
A: "I would use Docker Compose with a structure like this:

docker-compose.yml with separate networks for each tier (frontend, backend, database), use bind mounts for live code reloading, define all dependencies clearly, use .env files for configuration, and have separate compose files for dev/test environments. This allows developers to spin up the entire stack with one command while maintaining isolation and easy debugging."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

8. MULTI-STAGE BUILDS & OPTIMIZATION

Multi-stage builds allow you to create smaller, more secure production images by using multiple FROM statements.

Benefits:
• Smaller final image size (only runtime dependencies)
• Better security (no build tools in production)
• Single Dockerfile for build and runtime
• Layer caching still works

Documentation: https://docs.docker.com/build/building/multi-stage/

Basic Multi-stage Example:

# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]

Advanced Multi-stage (Go Application):

# Build stage
FROM golang:1.20-alpine AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Final stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /build/app .
EXPOSE 8080
CMD ["./app"]

Image Optimization Techniques:

1. Choose minimal base images:
   - alpine (5MB) over ubuntu (70MB)
   - distroless for even smaller
   - scratch for static binaries

Base Image Comparison: https://hub.docker.com/_/alpine

2. Minimize layers:
   - Combine RUN commands
   - Clean up in same layer
   
Bad:
RUN apt-get update
RUN apt-get install -y package
RUN apt-get clean

Good:
RUN apt-get update && apt-get install -y package && apt-get clean && rm -rf /var/lib/apt/lists/*

3. Order instructions by change frequency:
   - Least changing (dependencies) first
   - Most changing (source code) last
   - Maximizes cache hits

4. Use .dockerignore:
node_modules
.git
.env
*.log
README.md
.vscode
.idea
dist
build

5. Copy only what's needed:
COPY package*.json ./     # First
RUN npm install           # Cached if package.json unchanged
COPY . .                  # Last

Docker Image Security:

1. Scan for vulnerabilities:
docker scan myimage:tag

Tools:
• Trivy: https://github.com/aquasecurity/trivy
• Clair: https://github.com/quay/clair
• Snyk: https://snyk.io/product/container-vulnerability-management/

2. Run as non-root:
RUN addgroup -g 1000 appgroup && adduser -D -u 1000 -G appgroup appuser
USER appuser

3. Use specific versions:
FROM node:16.14.0-alpine3.15
# Not: FROM node:latest

4. Remove unnecessary packages:
RUN apk add --no-cache package

5. Use distroless images:
Documentation: https://github.com/GoogleContainerTools/distroless

FROM gcr.io/distroless/nodejs:16
COPY --from=builder /app /app
CMD ["/app/server.js"]

Best Practices Summary:
1. Multi-stage builds for production
2. Alpine or distroless base images
3. Combine RUN commands
4. Order by change frequency
5. Comprehensive .dockerignore
6. Run as non-root user
7. Use specific image tags
8. Regular vulnerability scanning
9. Minimize installed packages
10. Keep images small and focused

Learn more:
• Docker Security Best Practices: https://docs.docker.com/develop/security-best-practices/
• Image Optimization: https://docs.docker.com/develop/dev-best-practices/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART 2: KUBERNETES FUNDAMENTALS

11. WHAT IS KUBERNETES?

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

Key Features:
• Automatic bin packing - Optimal resource utilization
• Self-healing - Restarts failed containers, replaces nodes
• Horizontal scaling - Scale applications up/down based on metrics
• Service discovery and load balancing - Built-in DNS and load balancing
• Automated rollouts and rollbacks - Zero-downtime deployments
• Secret and configuration management - Secure credential storage
• Storage orchestration - Mount storage systems automatically
• Batch execution - Manage batch and CI workloads

Official Documentation: https://kubernetes.io/docs/home/
Kubernetes Concepts: https://kubernetes.io/docs/concepts/

Kubernetes vs Docker:
Kubernetes doesn't replace Docker - it orchestrates containers (Docker or containerd)

Docker: Container runtime
Kubernetes: Container orchestrator

Kubernetes Origins:
• Built by Google (based on Borg)
• Donated to CNCF in 2014
• Now industry standard

CNCF: https://www.cncf.io/

When to Use Kubernetes:
• Microservices architecture
• Need for high availability
• Auto-scaling requirements
• Multi-cloud or hybrid cloud
• Complex deployment patterns
• Large-scale applications

When NOT to Use Kubernetes:
• Simple monolithic applications
• Small teams without k8s expertise
• Minimal scaling needs
• Single-host deployments

Kubernetes Distributions:

1. Managed Services:
   • GKE (Google Kubernetes Engine): https://cloud.google.com/kubernetes-engine
   • EKS (AWS Elastic Kubernetes Service): https://aws.amazon.com/eks/
   • AKS (Azure Kubernetes Service): https://azure.microsoft.com/en-us/products/kubernetes-service
   • DigitalOcean Kubernetes: https://www.digitalocean.com/products/kubernetes

2. Self-managed:
   • kubeadm (official): https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/
   • kops: https://kops.sigs.k8s.io/
   • Kubespray: https://kubespray.io/

3. Local Development:
   • Minikube: https://minikube.sigs.k8s.io/
   • Kind (Kubernetes in Docker): https://kind.sigs.k8s.io/
   • k3s (Lightweight): https://k3s.io/
   • Docker Desktop: https://www.docker.com/products/docker-desktop/

4. Enterprise:
   • OpenShift (Red Hat): https://www.redhat.com/en/technologies/cloud-computing/openshift
   • Rancher: https://www.rancher.com/
   • Tanzu (VMware): https://tanzu.vmware.com/

Interview Scenario:
Q: "Explain Kubernetes in simple terms to a non-technical person."
A: "Kubernetes is like an intelligent traffic controller and resource manager for containerized applications. Imagine you have a fleet of shipping containers (your applications) and multiple warehouses (servers). Kubernetes automatically decides which containers go to which warehouses based on available space and resources, monitors them continuously, replaces damaged containers, scales up during busy times, and ensures everything runs smoothly without manual intervention. It's the autopilot for modern cloud applications."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

12. KUBERNETES ARCHITECTURE

Kubernetes Cluster = Control Plane + Worker Nodes

Architecture Diagram: https://kubernetes.io/docs/concepts/architecture/

CONTROL PLANE (Master Node)
Manages the cluster and makes global decisions

Components:
1. kube-apiserver
2. etcd
3. kube-scheduler
4. kube-controller-manager
5. cloud-controller-manager (optional)

WORKER NODES
Run the containerized applications (Pods)

Components:
1. kubelet
2. kube-proxy
3. Container Runtime (containerd/CRI-O)

Detailed Architecture Guide: https://devopscube.com/kubernetes-architecture-explained/
Video Tutorial (7 min): https://www.youtube.com/watch?v=vmCuNJiNzfg
Full Course (4 hours): https://www.youtube.com/watch?v=X48VuDVv0do

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

13. CONTROL PLANE COMPONENTS

The control plane makes global decisions about the cluster and detects/responds to cluster events.

Official Documentation: https://kubernetes.io/docs/concepts/overview/components/
Control Plane Deep Dive: https://spacelift.io/blog/kubernetes-control-plane

1. kube-apiserver

The API server is the front door to the Kubernetes control plane.

Key Responsibilities:
• Exposes the Kubernetes REST API
• Processes and validates REST requests
• Updates etcd with the state
• Authentication and authorization
• Admission control
• Single point of contact for all cluster operations

How it works:
1. kubectl sends request to API server
2. API server authenticates the request
3. API server authorizes (RBAC check)
4. Admission controllers validate/mutate
5. Request stored in etcd
6. Controllers/Schedulers watch for changes

Multiple API servers can run for HA (typically 3)

Commands:
kubectl cluster-info                  # Get API server URL
kubectl get --raw /api/v1             # Query API directly
kubectl api-resources                 # List all API resources
kubectl api-versions                  # List API versions

API Reference: https://kubernetes.io/docs/reference/kubernetes-api/

2. etcd

Distributed, consistent key-value store - the cluster's database.

Key Characteristics:
• Stores all cluster data
• Source of truth for cluster state
• Distributed and highly available
• Uses Raft consensus algorithm
• Typically 3 or 5 instances for HA
• Only API server talks to etcd

What's stored:
• Cluster configuration
• State of all objects (Pods, Services, etc.)
• Metadata
• Secrets (encrypted at rest)

Backup etcd regularly - it's your disaster recovery!

Backup command:
ETCDCTL_API=3 etcdctl snapshot save backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

Restore:
ETCDCTL_API=3 etcdctl snapshot restore backup.db --data-dir=/var/lib/etcd-restore

etcd Documentation: https://etcd.io/docs/
Kubernetes etcd Guide: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

3. kube-scheduler

Assigns Pods to Nodes based on constraints and available resources.

Scheduling Process:
1. Watch for newly created Pods with no assigned Node
2. Filtering - Find feasible Nodes (resource requirements, taints/tolerations, affinity)
3. Scoring - Rank feasible Nodes (spread pods, node affinity, resource balance)
4. Binding - Assign Pod to highest-scoring Node

Factors Considered:
• Resource requests (CPU, memory)
• Node selectors
• Node affinity/anti-affinity
• Pod affinity/anti-affinity
• Taints and tolerations
• Volume binding
• PodTopologySpread

Custom Schedulers:
You can write and use custom schedulers for specific requirements

Scheduler Configuration: https://kubernetes.io/docs/reference/scheduling/
Advanced Scheduling: https://kubernetes.io/docs/concepts/scheduling-eviction/

4. kube-controller-manager

Runs controller processes that regulate cluster state.

Built-in Controllers:
• Node Controller - Monitors node health
• Replication Controller - Maintains correct number of pods
• Endpoints Controller - Populates Endpoints objects
• Service Account & Token Controllers - Create default accounts/tokens
• Namespace Controller - Deletes resources in terminated namespaces
• Job Controller - Creates Pods for Jobs
• Deployment Controller - Manages ReplicaSets
• StatefulSet Controller - Manages StatefulSets
• DaemonSet Controller - Ensures pods run on all/selected nodes

Control Loop Pattern:
1. Watch desired state (from etcd via API server)
2. Observe current state
3. Make changes to reach desired state
4. Repeat

Example: Deployment Controller
- Watches Deployment objects
- Creates/updates ReplicaSets
- ReplicaSet controller creates/deletes Pods
- Scheduler assigns Pods to Nodes
- Kubelet starts containers

Controller Deep Dive: https://kubernetes.io/docs/concepts/architecture/controller/

5. cloud-controller-manager (Optional)

Integrates Kubernetes with cloud provider APIs.

Cloud-Specific Controllers:
• Node Controller - Check if node is deleted in cloud
• Route Controller - Set up routes in cloud infrastructure
• Service Controller - Create/update/delete cloud load balancers

Cloud Controller Documentation: https://kubernetes.io/docs/concepts/architecture/cloud-controller/

Interview Scenario:
Q: "Explain what happens when you run 'kubectl create deployment nginx --image=nginx' from start to finish."
A: "Complete flow:

1. kubectl sends POST request to kube-apiserver
2. API server authenticates and authorizes the request (RBAC)
3. Admission controllers validate/mutate the request
4. API server stores Deployment object in etcd
5. Deployment controller (watching API) detects new Deployment
6. Deployment controller creates a ReplicaSet
7. ReplicaSet is stored in etcd
8. ReplicaSet controller detects new ReplicaSet
9. ReplicaSet controller creates Pod(s) based on replicas
10. Pods are stored in etcd with status 'Pending'
11. Scheduler watches for Pending pods
12. Scheduler evaluates nodes and assigns Pod to best node
13. Scheduler updates Pod with nodeName in etcd
14. Kubelet on that node watches API server
15. Kubelet sees Pod assigned to its node
16. Kubelet tells container runtime to pull nginx image
17. Container runtime pulls image and starts container
18. Kubelet updates Pod status to 'Running' via API server
19. Status stored in etcd

This entire process typically takes 1-5 seconds for a simple deployment."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

14. NODE COMPONENTS

Worker nodes host the Pods and execute containers.

Node Components Documentation: https://kubernetes.io/docs/concepts/architecture/nodes/

1. kubelet

Agent running on each node - ensures containers are running in Pods.

Key Responsibilities:
• Registers node with API server
• Watches API server for Pods assigned to its node
• Ensures Pod containers are running and healthy
• Reports Pod and node status to API server
• Executes liveness and readiness probes
• Mounts volumes
• Collects and reports resource metrics

kubelet communicates with container runtime via CRI (Container Runtime Interface)

kubelet Configuration:
Configuration file: /var/lib/kubelet/config.yaml

Common flags:
--pod-manifest-path         # Static pod directory
--cluster-dns               # DNS server IP
--cluster-domain            # Cluster domain (default: cluster.local)
--kubeconfig                # Auth config
--container-runtime-endpoint  # CRI socket

Static Pods:
kubelet can run pods directly from manifest files in a directory (used for control plane components)

kubelet Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/

2. kube-proxy

Network proxy running on each node - implements Service networking.

Key Responsibilities:
• Maintains network rules on nodes
• Forwards traffic to correct Pods
• Load balances across Pod replicas
• Handles Service ClusterIP routing

Proxy Modes:

a) iptables mode (default)
   • Uses iptables rules for routing
   • Good for most use cases
   • Can handle thousands of Services
   • Random load balancing

b) IPVS mode
   • Uses Linux IPVS (IP Virtual Server)
   • Better performance at scale
   • More load balancing algorithms
   • Recommended for large clusters

c) userspace mode (legacy)
   • Older, slower
   • Rarely used

kube-proxy in IPVS mode:
kube-proxy --proxy-mode=ipvs

Load balancing algorithms in IPVS:
• rr (round-robin)
• lc (least connection)
• dh (destination hashing)
• sh (source hashing)
• sed (shortest expected delay)
• nq (never queue)

kube-proxy Documentation: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/
Service Networking: https://kubernetes.io/docs/concepts/services-networking/service/

3. Container Runtime

Software responsible for running containers.

Supported Runtimes:

a) containerd (Recommended)
   • Industry-standard runtime
   • Lightweight and efficient
   • CNCF graduated project
   • Docker uses containerd internally
   • https://containerd.io/

b) CRI-O
   • Lightweight, OCI-compliant
   • Built specifically for Kubernetes
   • https://cri-o.io/

c) Docker (via containerd)
   • Docker Engine support removed in K8s 1.24
   • Use containerd directly instead

Container Runtime Interface (CRI):
Standardized interface between kubelet and runtime

CRI Documentation: https://kubernetes.io/docs/concepts/architecture/cri/

Node Status:

Check node status:
kubectl get nodes
kubectl describe node <node-name>

Node Conditions:
• Ready - Node is healthy and ready to accept Pods
• MemoryPressure - Node has memory pressure
• DiskPressure - Node has disk pressure
• PIDPressure - Too many processes
• NetworkUnavailable - Network not configured

Node Information:
• Addresses (internal/external IP, hostname)
• Capacity (CPU, memory, pods)
• Allocatable resources
• System info (OS, kernel, container runtime)

Interview Scenario:
Q: "How does kube-proxy enable Service networking in Kubernetes?"
A: "kube-proxy runs as a DaemonSet on every node and maintains network rules that enable Service abstraction:

1. When a Service is created, it gets a ClusterIP
2. kube-proxy watches the API server for Service and Endpoint changes
3. Based on the proxy mode (iptables/IPVS), it configures network rules
4. In iptables mode, it creates iptables rules that intercept traffic to the ClusterIP
5. These rules randomly distribute traffic to backend Pod IPs
6. When Pods are added/removed, Endpoints are updated
7. kube-proxy immediately updates the rules

For example, if a Service has ClusterIP 10.96.0.1:80 with 3 backend Pods, kube-proxy creates rules that forward any traffic to 10.96.0.1:80 to one of the 3 Pod IPs randomly. This provides load balancing and service discovery without applications needing to know individual Pod IPs.

In production, I've used IPVS mode for large clusters (1000+ services) because it's more efficient and offers better load balancing algorithms like least-connection and source-hashing."



━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PART 3: KUBERNETES WORKLOADS

16. PODS

Pods are the smallest deployable units in Kubernetes - a group of one or more containers.

Pod Characteristics:
• Atomic unit of scheduling
• Share network namespace (same IP)
• Share storage volumes
• Ephemeral (not durable)
• Typically one container per Pod (sidecar pattern for multiple)

Pod Documentation: https://kubernetes.io/docs/concepts/workloads/pods/

Basic Pod YAML:

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80

Pod with Multiple Containers (Sidecar Pattern):

apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: main-app
    image: myapp:1.0
    ports:
    - containerPort: 8080
  - name: log-sidecar
    image: fluent/fluentd:latest
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
  volumes:
  - name: log-volume
    emptyDir: {}

Pod Lifecycle:

Phases:
1. Pending - Accepted but not yet scheduled/started
2. Running - Bound to node and at least one container running
3. Succeeded - All containers terminated successfully
4. Failed - All containers terminated, at least one failed
5. Unknown - State cannot be obtained

Container States:
• Waiting - Pulling image, waiting for resources
• Running - Executing
• Terminated - Finished or failed

Init Containers:
Run before main containers, must complete successfully

spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do sleep 2; done;']
  containers:
  - name: main-app
    image: myapp:1.0

Init Containers Guide: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

Pod Resource Requests & Limits:

resources:
  requests:      # Minimum guaranteed
    memory: "64Mi"
    cpu: "250m"
  limits:        # Maximum allowed
    memory: "128Mi"
    cpu: "500m"

CPU:
• 1 CPU = 1000 millicores (m)
• 500m = 0.5 CPU

Memory:
• Units: Ki, Mi, Gi, Ti (1024-based)
• Or: K, M, G, T (1000-based)

Resource Management: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

Probes (Health Checks):

1. Liveness Probe
   • Checks if container is alive
   • Restarts container if fails

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

2. Readiness Probe
   • Checks if container ready to accept traffic
   • Removes from Service endpoints if fails

readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

3. Startup Probe
   • For slow-starting containers
   • Disables liveness/readiness until succeeds

startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

Probe Types:
• httpGet - HTTP GET request
• tcpSocket - TCP connection
• exec - Execute command

Probe Configuration: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

Pod Commands:

kubectl run nginx --image=nginx                    # Create Pod
kubectl get pods                                   # List Pods
kubectl get pods -o wide                          # More details
kubectl describe pod nginx                         # Detailed info
kubectl logs nginx                                 # View logs
kubectl logs nginx -f                             # Follow logs
kubectl logs nginx -c container-name              # Specific container
kubectl exec -it nginx -- /bin/bash               # Enter container
kubectl delete pod nginx                           # Delete Pod
kubectl get pods --watch                          # Watch changes
kubectl top pod nginx                             # Resource usage

Pod Networking:
• Each Pod gets unique IP
• Containers in same Pod communicate via localhost
• Pods communicate directly using Pod IPs
• CNI plugin handles networking

Pod Security:

securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  capabilities:
    add: ["NET_ADMIN"]
    drop: ["ALL"]
  readOnlyRootFilesystem: true

Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/

Best Practices:
1. One container per Pod (except sidecars)
2. Always set resource requests and limits
3. Use liveness and readiness probes
4. Run as non-root user
5. Use specific image tags (not latest)
6. Set security context
7. Label pods appropriately

Interview Scenario:
Q: "What's the difference between liveness and readiness probes?"
A: "Both are health checks but serve different purposes:

Liveness Probe:
- Determines if container is alive/healthy
- If fails, kubelet restarts the container
- Use for detecting deadlocks or hung applications
- Example: Application is running but stuck in infinite loop

Readiness Probe:
- Determines if container is ready to serve traffic
- If fails, Pod removed from Service endpoints (no traffic sent)
- Container keeps running, not restarted
- Use for warming up, loading data, waiting for dependencies
- Example: Application started but still loading configuration from database

Real scenario: I had a Java microservice that took 60 seconds to initialize. I used:
- startupProbe: Allow 60 seconds for initialization
- readinessProbe: Check if /health returns 200 (means loaded and ready)
- livenessProbe: Check if /alive returns 200 (means not deadlocked)

This prevented traffic from hitting the service before it was ready, and automatically restarted if it became unresponsive."



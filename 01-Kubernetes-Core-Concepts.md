# PART 1: DOCKER FUNDAMENTALS & ADVANCED

## 1. INTRODUCTION TO CONTAINERIZATION

### What is Containerization?

Containerization is a lightweight form of virtualization that packages an application and its dependencies together in an isolated environment called a container. Unlike traditional virtual machines (VMs), containers share the host OS kernel, making them more efficient and faster to start. [page:4]

### Key Differences: Containers vs Virtual Machines

| Aspect          | Virtual Machines                    | Containers                  |
|----------------|--------------------------------------|-----------------------------|
| Size           | GBs (includes full OS)              | MBs (shares host kernel)    |
| Startup Time   | Minutes                             | Seconds                     |
| Resource Usage | Heavy (hypervisor overhead)         | Lightweight                 |
| Isolation      | Complete OS-level isolation         | Process-level isolation     |
| Portability    | Less portable                       | Highly portable             |
| Performance    | Slower                              | Near-native performance     | [page:4]

### Benefits of Containerization

- Consistency across environments (dev, test, prod)  
- Faster deployment and scaling  
- Efficient resource utilization  
- Microservices architecture enablement  
- Version control for infrastructure  
- Simplified dependency management [page:4]

Learn more:

- Docker Overview: https://docs.docker.com/get-started/overview/  
- Containers vs VMs: https://www.docker.com/resources/what-container/ [page:4]

### Interview Scenario

**Q:** Explain the difference between containers and virtual machines.  

**A:** Containers virtualize the operating system, sharing the host kernel, while VMs virtualize hardware, each running a full OS. This makes containers lighter (MBs vs GBs), faster to start (seconds vs minutes), and more resource-efficient, while VMs provide stronger isolation. In practice, containers are typically used for microservices deployments needing fast scaling, and VMs for workloads needing complete OS isolation. [page:4]

---

## 2. DOCKER ARCHITECTURE & COMPONENTS

### Docker Architecture Overview

Docker uses a client-server architecture with three main components: [page:4]

1. **Docker Client (docker CLI)**  
   - Command-line interface for users  
   - Sends commands to Docker daemon via REST API  
   - Can communicate with remote daemons  
   - Common commands: `docker build`, `docker run`, `docker push`  

2. **Docker Daemon (dockerd)**  
   - Background service running on the host  
   - Manages Docker objects: images, containers, networks, volumes  
   - Listens to Docker API requests  
   - Can communicate with other daemons for distributed systems  

3. **Docker Registry**  
   - Stores Docker images  
   - Docker Hub: public registry (default)  
   - Private registries: AWS ECR, Google GCR, Azure ACR, Harbor, Artifactory  
   - Common commands: `docker pull`, `docker push` [page:4]

### Docker Objects

**Images**

- Read-only templates with instructions for creating containers  
- Built from Dockerfile  
- Layered filesystem (each instruction creates a layer)  
- Layers are cached and reused  
- Base images from Docker Hub (for example: `ubuntu`, `alpine`, `nginx`) [page:4]

**Containers**

- Runnable instances of images  
- Isolated from other containers and host  
- Can be started, stopped, moved, deleted  
- Defined by image and configuration options  
- Ephemeral by default (data lost when removed) [page:4]

**Networks**

- Enable communication between containers  
- Types: bridge, host, overlay, macvlan, none  
- Network drivers for different use cases [page:4]

**Volumes**

- Persistent data storage  
- Managed by Docker  
- Survive container deletion  
- Can be shared between containers [page:4]

### Docker Engine Components

1. **containerd**  
   - Industry-standard container runtime  
   - Manages container lifecycle  
   - Handles image transfer and storage  
   - Supervises container execution  

2. **runc**  
   - Low-level container runtime  
   - OCI-compliant  
   - Creates and runs containers  

3. **shim**  
   - Keeps STDIN/STDOUT open  
   - Reports exit status  
   - Allows daemon restart without killing containers [page:4]

Learn more:

- Docker Architecture: https://docs.docker.com/get-started/overview/#docker-architecture  
- containerd: https://containerd.io/  
- OCI Runtime 

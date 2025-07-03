# 1.1 What is a Container

## Introduction to Containers
A container is a self-contained, ready-to-run application package that includes everything needed to run an application. Unlike virtual machines, containers share the host system's kernel, making them more lightweight and efficient.

## Container Architecture

### Key Components
1. **Container Image**
   - Read-only template containing application code and dependencies
   - Includes the application and all its dependencies
   - Based on a layered filesystem for efficient storage

2. **Container Runtime**
   - Software responsible for running containers
   - Manages container lifecycle (start, stop, pause, etc.)
   - Examples: Docker Engine, containerd, CRI-O

3. **Host Kernel**
   - Containers share the host OS kernel
   - Uses kernel features like namespaces and cgroups for isolation
   - No separate OS kernel per container (unlike VMs)

## Containers vs Virtual Machines

### Virtual Machines
- Each VM runs its own full OS
- Requires a hypervisor
- Higher resource overhead
- Slower startup time
- Strong isolation

### Containers
- Share the host OS kernel
- Use container runtime
- Lightweight and fast
- Quick startup time
- Process-level isolation

## Linux Kernel Features

### Namespaces
- Provide process isolation
- Types:
  - PID (Process ID)
  - Network
  - Mount
  - IPC (Inter-Process Communication)
  - UTS (Hostname)
  - User

### Control Groups (cgroups)
- Manage and limit resource usage
- Control:
  - CPU
  - Memory
  - Disk I/O
  - Network bandwidth

## Container Standards (OCI)
- **Open Container Initiative (OCI)**
  - Industry standards for container formats and runtime
  - Defines image specification (image-spec)
  - Defines runtime specification (runtime-spec)
  - Ensures compatibility between different container runtimes

## Container Solutions

### Docker
- Most popular container platform
- Includes:
  - Container runtime
  - Image format
  - Build tools (Dockerfile)
  - Image distribution (Docker Hub)

### Podman
- Red Hat's daemonless container engine
- Docker-compatible CLI
- Rootless containers
- Uses CRI-O as default runtime

### Other Solutions
- **LXC (Linux Containers)**: Lightweight VMs alternative
- **rkt**: Security-focused container runtime
- **containerd**: Industry-standard container runtime

## Why Containers for Kubernetes?
- Lightweight and portable
- Consistent environments
- Efficient resource usage
- Fast deployment and scaling
- Isolation and security

## Key Takeaways
- Containers package applications and dependencies together
- Share the host OS kernel
- Use Linux namespaces and cgroups for isolation
- Multiple container runtimes available
- OCI standards ensure compatibility
- Docker is the most common choice for Kubernetes

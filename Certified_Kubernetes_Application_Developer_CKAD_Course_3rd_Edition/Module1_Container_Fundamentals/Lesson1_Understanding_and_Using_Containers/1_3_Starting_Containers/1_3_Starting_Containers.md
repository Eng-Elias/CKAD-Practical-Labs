# 1.3 Starting Containers

## Container Runtimes Overview

### Docker
- **Ubuntu Support**: Recommended for Docker installations
- **Installation**:
  ```bash
  # Update package index
  sudo apt-get update
  
  # Install required packages
  sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common

  # Add Docker's official GPG key
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  
  # Add Docker repository
  sudo apt-key fingerprint 0EBFCD88
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  
  # Install Docker CE
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io

  # Verify installation
  sudo docker run hello-world
  ```

### Podman (RHEL 8+ Alternative)
- Default container runtime on RHEL 8+
- No daemon required
- Installation:
  ```bash
  # Install container-tools
  sudo dnf install -y @container-tools
  ```

## Running Your First Container

### Basic Container Execution
```bash
# Run a simple hello-world container
docker run hello-world
```

### Running in Detached Mode
```bash
# Run a container in the background
docker run -d nginx
```

### Port Forwarding
```bash
# Map host port 8080 to container port 80
docker run -d -p 8080:80 --name my-nginx nginx
```

### Volume Mounting
```bash
# Create a directory and file
echo "Hello from Docker" > index.html

# Mount local directory to container
mkdir -p /var/www/html
echo "Hello from Docker" | sudo tee /var/www/html/index.html
docker run -d -p 8080:80 -v /var/www/html:/usr/share/nginx/html --name my-web nginx
```

## Container Lifecycle Management

### Starting and Stopping
```bash
# Start a container
docker start <container_id>

# Stop a container
docker stop <container_id>

# Restart a container
docker restart <container_id>
```

### Removing Containers
```bash
# Remove a stopped container
docker rm <container_id>

# Force remove a running container
docker rm -f <container_id>
```

## Interactive Containers

### Running an Interactive Shell
```bash
# Start an interactive shell in a container
docker run -it ubuntu /bin/bash

# To exit without stopping the container
# Press Ctrl+P then Ctrl+Q
```

### Connecting to a Running Container
```bash
# Open a shell in a running container
docker exec -it <container_id> /bin/bash
```

## Container Inspection

### Viewing Running Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a
```

### Viewing Container Logs
```bash
# View container logs
docker logs <container_id>

# Follow log output
docker logs -f <container_id>
```

### Inspecting Container Details
```bash
# View detailed container information
docker inspect <container_id>

# Get container IP address
docker inspect -f '{{.NetworkSettings.IPAddress}}' <container_id>
```

## Best Practices

1. **Naming Containers**
   - Always use `--name` for better management
   - Example: `docker run --name myapp nginx`

2. **Resource Management**
   - Set memory and CPU limits
   - Example: `docker run -m 512m --cpus=1 nginx`

3. **Cleanup**
   - Remove unused containers to save disk space
   - Use `docker container prune` to clean up stopped containers

4. **Security**
   - Run containers as non-root when possible
   - Use read-only filesystems where applicable
   - Example: `docker run --read-only -v /path/to/writeable:/data nginx`

## Common Issues and Solutions

### Port Already in Use
```bash
# Find and stop the process using the port
sudo lsof -i :8080
sudo kill <process_id>
```

### Container Exits Immediately
- Check logs: `docker logs <container_id>`
- Ensure the main process stays running
- Use `-d` for long-running processes

### Permission Issues
- Add your user to the docker group:
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker  # Apply group changes
  ```

## Next Steps
- Learn about container networking
- Explore container orchestration with Kubernetes
- Study container image creation with Dockerfiles

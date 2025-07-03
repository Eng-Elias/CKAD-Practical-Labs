# 1.4 Managing Containers

## Container Lifecycle Commands

### Viewing Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a
```

### Starting and Stopping Containers
```bash
# Start a container
docker start <container_id_or_name>

# Stop a container (graceful shutdown)
docker stop <container_id_or_name>

# Force stop a container (immediate)
docker kill <container_id_or_name>

# Restart a container
docker restart <container_id_or_name>
```

### Removing Containers
```bash
# Remove a stopped container
docker rm <container_id_or_name>

# Force remove a running container
docker rm -f <container_id_or_name>

# Remove all stopped containers
docker container prune
```

## Container Inspection

### Basic Inspection
```bash
# View detailed container information
docker inspect <container_id_or_name>

# Filter specific information using JSON path
docker inspect -f '{{.NetworkSettings.IPAddress}}' <container_name>
docker inspect -f '{{.State.Pid}}' <container_name>
```

### Common JSON Path Filters
- Network settings: `{{.NetworkSettings.IPAddress}}`
- Process ID: `{{.State.Pid}}`
- Container status: `{{.State.Status}}`
- Mounts: `{{.Mounts}}`
- Environment variables: `{{.Config.Env}}`

## Process Management

### Viewing Container Processes
```bash
# View processes in a container
docker top <container_id_or_name>

# Execute a command in a running container
docker exec -it <container_id_or_name> <command>

# Open an interactive shell in a container
docker exec -it <container_id_or_name> /bin/bash
# or for minimal images
docker exec -it <container_id_or_name> /bin/sh
```

### Host System Process View
```bash
# View container processes from host
ps aux | grep -i docker
# or
ps -ef | grep -i containerd
```

## Resource Management

### Viewing Resource Usage
```bash
# Show container resource usage statistics
docker stats

# Show running processes in a container
docker top <container_id_or_name>
```

### Resource Limits
```bash
# Run container with memory and CPU limits
docker run -it --memory="1g" --cpus="1.0" ubuntu
```

## Best Practices

1. **Naming Containers**
   - Always use `--name` for better management
   - Example: `docker run --name myapp nginx`

2. **Cleanup**
   - Remove unused containers to save disk space
   - Use `docker system prune` to clean up unused containers, networks, and images

3. **Resource Management**
   - Set appropriate memory and CPU limits
   - Monitor container resource usage

4. **Security**
   - Run containers as non-root when possible
   - Keep containers minimal by only including necessary packages

## Troubleshooting

### Common Issues
1. **Container exits immediately**
   - Check logs: `docker logs <container_id>`
   - Run interactively: `docker run -it <image> /bin/sh`

2. **Port conflicts**
   ```bash
   # Find process using a port
   sudo lsof -i :<port>
   # or
   sudo netstat -tulpn | grep :<port>
   ```

3. **Permission issues**
   - Ensure your user is in the docker group:
     ```bash
     sudo usermod -aG docker $USER
     newgrp docker
     ```

## Understanding Container Processes
- Containers appear as regular processes on the host system
- Managed by container runtimes like `containerd`
- Each container has its own PID namespace
- Use `ps` on the host to see container processes
- Look for `containerd-shim` in process list to identify container processes
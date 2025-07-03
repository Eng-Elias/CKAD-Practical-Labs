# 1.6 Understanding Container Logging

## Introduction to Container Logs

Containers handle logging differently from traditional applications. By default, container applications write their output to standard output (stdout) and standard error (stderr) streams, which are then captured by the container runtime.

### Key Points About Container Logging:
- Container applications don't connect directly to the terminal's stdout/stderr
- Logs are captured by the container runtime (Docker, containerd, etc.)
- You need special commands to access these logs

## Working with Docker Logs

### Viewing Logs of Running Containers
```bash
# Basic log viewing
docker logs <container_name_or_id>

# Follow logs in real-time (similar to tail -f)
docker logs -f <container_name_or_id>

# Show last N lines
docker logs --tail=100 <container_name_or_id>

# Show logs with timestamps
docker logs -t <container_name_or_id>
```

### Understanding Container States
- **Running**: Container is currently active
- **Exited (0)**: Container stopped successfully (nothing to investigate)
- **Exited (non-zero)**: Container stopped with an error (investigate with logs)

## Practical Example: Troubleshooting a MariaDB Container

### Scenario: Container Fails to Start
1. **Run a container in detached mode**
   ```bash
   docker run -d --name mydb mariadb
   ```

2. **Check container status**
   ```bash
   docker ps -a
   ```
   - If status shows "Exited (1)", there's an error

3. **View container logs**
   ```bash
   docker logs mydb
   ```
   This will show why the container failed to start (e.g., missing environment variables)

### Common Issues and Solutions

#### 1. Missing Required Environment Variables
```bash
# Error in logs: Database is uninitialized and password option is not specified
# Solution: Provide required environment variables

docker run -d \
  --name mydb \
  -e MYSQL_ROOT_PASSWORD=mysecretpassword \
  mariadb
```

#### 2. Container Name Already in Use
```bash
# Error: The container name "/mydb" is already in use
# Solution: Remove the existing container first

docker rm mydb
docker run -d --name mydb mariadb
```

#### 3. Incorrect Command Order
```bash
# Incorrect (options after image name are passed to the container)
docker run -d mariadb -e MYSQL_ROOT_PASSWORD=password

# Correct (options before image name are for Docker)
docker run -d -e MYSQL_ROOT_PASSWORD=password mariadb
```

## Best Practices for Container Logging

1. **Always Check Logs for Failed Containers**
   - Use `docker logs` when a container exits unexpectedly
   - Look for error messages that explain the failure

2. **Use Meaningful Container Names**
   - Makes it easier to identify containers when checking logs
   - Example: `--name database-prod` instead of auto-generated names

3. **Configure Log Rotation**
   ```bash
   docker run --log-opt max-size=10m --log-opt max-file=3 myapp
   ```

4. **Consider Log Drivers**
   - Docker supports various log drivers (json-file, syslog, journald, etc.)
   - Configure based on your logging infrastructure

## Kubernetes Connection

In Kubernetes, the equivalent command is:
```bash
kubectl logs <pod_name> -n <namespace>
```

This works similarly to Docker logs but operates at the Kubernetes level, which adds complexity with:
- Multiple containers per pod
- Cluster-wide logging
- Log aggregation in cloud environments

## Common Docker Logs Options

| Option | Description |
|--------|-------------|
| `-f, --follow` | Follow log output |
| `--tail` | Number of lines to show from the end |
| `--since` | Show logs since timestamp or relative time |
| `--until` | Show logs before a timestamp |
| `-t, --timestamps` | Show timestamps |
| `--details` | Show extra details |

## Troubleshooting Tips

1. **Container Exited Immediately**
   - Check logs with `docker logs <container>`
   - Look for errors in the application startup

2. **No Logs Available**
   - The container might not be writing to stdout/stderr
   - Check if logs are written to files inside the container

3. **Logs Too Verbose**
   - Filter logs using `grep`: `docker logs <container> | grep ERROR`
   - Use `--tail` to limit output

Remember that proper logging is crucial for debugging and monitoring containerized applications. Always ensure your applications are configured to log to stdout/stderr when running in containers.
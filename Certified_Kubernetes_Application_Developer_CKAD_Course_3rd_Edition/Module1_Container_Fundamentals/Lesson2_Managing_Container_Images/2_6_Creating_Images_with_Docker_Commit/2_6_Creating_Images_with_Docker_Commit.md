# 2.6 Creating Images with Docker Commit

## Introduction to `docker commit`

The `docker commit` command creates a new image from a container's changes. While not the recommended approach for production, it's useful for quick testing and debugging.

## Basic Workflow

1. Start a container from a base image
2. Make changes inside the container
3. Commit the container to create a new image

## Step-by-Step Guide

### 1. Start a Container
```bash
docker run -it --name mycontainer ubuntu:20.04 /bin/bash
```

### 2. Make Changes Inside Container
```bash
# Inside container
apt update
apt install -y curl
curl --version
```

### 3. Commit the Container
In a new terminal:
```bash
docker commit \
  --author="Your Name <your.email@example.com>" \
  --message="Added curl to Ubuntu" \
  mycontainer \
  my-ubuntu-curl:1.0
```

### 4. Verify the New Image
```bash
docker images | grep my-ubuntu-curl
docker history my-ubuntu-curl:1.0
```

## Common Use Cases

### 1. Saving Debugging State
```bash
# After debugging a failing container
docker commit my-failing-container debug-image:1.0
```

### 2. Creating Quick Prototypes
```bash
# After setting up a development environment
docker commit my-dev-env dev-environment:latest
```

### 3. Creating Backups
```bash
# Before making major changes
docker commit my-running-app my-app-backup:$(date +%Y%m%d)
```

## Inspecting Changes

### View Filesystem Changes
```bash
docker diff mycontainer
```
Output format:
- `A` - Added
- `D` - Deleted
- `C` - Changed

### View Container Configuration
```bash
docker inspect mycontainer
```

## Saving and Loading Images

### Save Image to Tar Archive
```bash
docker save -o my-ubuntu-curl.tar my-ubuntu-curl:1.0
```

### Load Image from Tar Archive
```bash
docker load -i my-ubuntu-curl.tar
```

## Limitations and Drawbacks

### 1. Lack of Reproducibility
- No record of how changes were made
- Difficult to audit or modify later

### 2. Larger Image Size
- Includes all intermediate files and caches
- No layer optimization

### 3. Security Risks
- May contain sensitive information
- Hard to track installed packages and versions

## Best Practices (When You Must Use It)

### 1. Document Changes
```bash
docker commit \
  --change="LABEL description='Ubuntu with curl installed'" \
  --change="LABEL maintainer='dev@example.com'" \
  mycontainer \
  my-ubuntu-curl:1.0
```

### 2. Clean Up Before Committing
```bash
# Inside container
apt clean
rm -rf /var/lib/apt/lists/*
rm -rf /tmp/*
```

### 3. Use Meaningful Tags
```bash
docker commit mycontainer myapp:$(date +%Y%m%d)-debug
```

## Practical Example: Debugging a Web Server

### 1. Start a Web Server
```bash
docker run -d --name webserver nginx
docker exec -it webserver /bin/bash

# Inside container:
apt update
apt install -y vim curl
# Make some configuration changes...
```

### 2. Commit the Debugged Version
```bash
docker commit webserver nginx-debug:1.0
```

### 3. Run the Debugged Version
```bash
docker run -d -p 8080:80 --name debugged-webserver nginx-debug:1.0
```

## Converting to Dockerfile

For better maintainability, convert your committed image to a Dockerfile:

1. Get the history:
   ```bash
   docker history --no-trunc my-ubuntu-curl:1.0
   ```

2. Create a Dockerfile based on the commands shown

## When to Avoid `docker commit`

- Production deployments
- Team environments
- CI/CD pipelines
- When you need to update the image later

## Next Steps
- Learn about Dockerfile best practices
- Explore multi-stage builds
- Understand image optimization techniques
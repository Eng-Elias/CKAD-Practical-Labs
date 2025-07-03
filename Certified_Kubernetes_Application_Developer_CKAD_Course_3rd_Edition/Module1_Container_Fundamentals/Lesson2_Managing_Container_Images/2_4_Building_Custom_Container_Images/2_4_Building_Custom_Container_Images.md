# 2.4 Building Custom Container Images

## Introduction to Custom Images

Custom container images allow you to package applications and their dependencies into portable, reproducible units. This lesson covers the fundamentals of creating optimized container images.

## Image Terminology

### Parent and Child Images
- **Parent Image**: Base image that your image builds upon
- **Child Image**: Your customized image that inherits from a parent
- **Layers**: Each instruction in a Dockerfile creates a new layer

### Dockerfile vs Containerfile
- **Dockerfile**: Original filename used by Docker
- **Containerfile**: More generic term (used by Podman and other OCI-compliant tools)
- Functionally identical, but some tools may prefer one over the other

## Building Blocks of a Custom Image

### 1. Base Image Selection
```dockerfile
# Official minimal image
FROM alpine:3.14

# Official full-featured image
FROM ubuntu:20.04

# Scratch (empty) image
FROM scratch
```

### 2. Metadata and Labels
```dockerfile
LABEL maintainer="your.email@example.com"
LABEL version="1.0"
LABEL description="Custom Nginx image"
```

### 3. Environment Variables
```dockerfile
ENV NGINX_VERSION=1.21.0
ENV APP_HOME=/app
```

### 4. Working Directory
```dockerfile
WORKDIR /app
```

### 5. Copying Files
```dockerfile
# Copy single file
COPY package.json .

# Copy directory
COPY src/ /app/src/

# Copy with wildcards
COPY *.txt /data/
```

### 6. Running Commands
```dockerfile
# Single command
RUN apt update && apt install -y nginx

# Multi-line command
RUN apt update && \
    apt install -y \
    nginx \
    curl \
    && rm -rf /var/lib/apt/lists/*
```

## Building the Image

### Basic Build Command
```bash
docker build -t myapp:1.0 .
```

### Build with No Cache
```bash
docker build --no-cache -t myapp:1.0 .
```

### Specifying Dockerfile Location
```bash
docker build -f Dockerfile.prod -t myapp:prod .
```

## Best Practices for Building Images

### 1. Use Multi-Stage Builds
```dockerfile
# Build stage
FROM node:14 as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
```

### 2. Minimize Layers
- Combine related commands with `&&`
- Clean up in the same layer
- Use `\` for better readability

### 3. Security Considerations
- Run as non-root user
- Use `.dockerignore`
- Scan for vulnerabilities
- Keep images updated

## Practical Example: Building a Python Application

### Directory Structure
```
myapp/
├── app.py
├── requirements.txt
└── Dockerfile
```

### app.py
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### requirements.txt
```
flask==2.0.1
```

### Dockerfile
```dockerfile
# Build stage
FROM python:3.9-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Runtime stage
FROM python:3.9-slim
WORKDIR /app

# Copy only what's needed
COPY --from=builder /root/.local /root/.local
COPY app.py .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

# Run as non-root user
RUN useradd -m myuser && chown -R myuser /app
USER myuser

EXPOSE 5000
CMD ["python", "app.py"]
```

## Building and Running

```bash
# Build the image
docker build -t myapp:1.0 .

# Run the container
docker run -p 5000:5000 myapp:1.0
```

## Common Issues and Solutions

### 1. Build Context Too Large
- Use `.dockerignore`
- Only copy necessary files
- Consider using multi-stage builds

### 2. Permission Issues
- Set proper file permissions
- Use appropriate user context
- Handle volume permissions

### 3. Large Image Size
- Use smaller base images
- Clean up package caches
- Remove unnecessary files

## Next Steps
- Explore multi-architecture builds
- Learn about build arguments
- Understand image optimization techniques
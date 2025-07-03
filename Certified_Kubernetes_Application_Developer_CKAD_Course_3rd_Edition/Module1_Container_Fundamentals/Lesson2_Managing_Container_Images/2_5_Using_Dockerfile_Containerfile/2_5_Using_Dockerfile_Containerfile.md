# 2.5 Using Dockerfile/Containerfile

## Introduction

Dockerfile (or Containerfile) is a text document that contains all the commands a user could call on the command line to assemble an image. This lesson dives deep into creating efficient and secure Dockerfiles.

## Basic Structure

### File Naming Conventions
- `Dockerfile` - Default filename recognized by Docker
- `Dockerfile.custom` - For alternate configurations (use with `-f` flag)
- `Containerfile` - Used by Podman and other OCI-compliant tools

### File Format
```dockerfile
# Comment
INSTRUCTION arguments
```

## Essential Instructions

### 1. FROM
Specifies the base image:
```dockerfile
# Official image
FROM ubuntu:20.04

# With tag
FROM nginx:1.21.0

# With digest for immutability
FROM nginx@sha256:1234...
```

### 2. RUN
Executes commands in a new layer:
```dockerfile
# Shell form (uses /bin/sh -c)
RUN apt-get update && apt-get install -y curl

# Exec form (recommended)
RUN ["/bin/bash", "-c", "echo hello"]
```

### 3. CMD vs ENTRYPOINT

#### CMD
Provides default command/arguments:
```dockerfile
# Shell form
CMD echo "Hello, World!"

# Exec form (preferred)
CMD ["nginx", "-g", "daemon off;"]

# As default parameters to ENTRYPOINT
CMD ["--help"]
```

#### ENTRYPOINT
Configures the container to run as an executable:
```dockerfile
# Shell form (not recommended with CMD)
ENTRYPOINT /app/start.sh

# Exec form (recommended)
ENTRYPOINT ["/app/start.sh"]

# With CMD as parameters
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

### 4. COPY vs ADD

#### COPY
```dockerfile
# Basic copy
COPY src/ /app/

# Copy with permissions
COPY --chown=user:group src/ /app/
```

#### ADD
```dockerfile
# Copy and extract tarball
ADD app.tar.gz /app/

# Copy from URL
ADD https://example.com/file.txt /tmp/
```

> **Note**: Prefer COPY over ADD unless you need the additional features of ADD.

## Advanced Features

### Multi-Stage Builds
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

### Build Arguments
```dockerfile
ARG VERSION=latest
FROM ubuntu:${VERSION}

# Build with: docker build --build-arg VERSION=20.04 .
```

### Healthcheck
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

## Best Practices

### 1. Order Instructions
1. Install system dependencies
2. Install application dependencies
3. Copy application code
4. Set up runtime configuration

### 2. Layer Optimization
```dockerfile
# Bad - creates multiple layers
RUN apt update
RUN apt install -y package
RUN rm -rf /var/lib/apt/lists/*

# Good - single layer
RUN apt update && \
    apt install -y package && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Security
```dockerfile
# Run as non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Don't run as root
# Don't store secrets in the image
# Use multi-stage builds to reduce attack surface
```

## Complete Example

```dockerfile
# Build stage
FROM node:14 as builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Runtime stage
FROM node:14-slim

WORKDIR /app

# Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser appuser && \
    chown -R appuser:appuser /app

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Set environment variables
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# Run as non-root user
USER appuser

# Command to run
CMD ["node", "app.js"]
```

## Common Pitfalls

### 1. Caching Issues
```dockerfile
# Bad - invalidates cache for all following steps
COPY . .
RUN npm install

# Good - leverages layer caching
COPY package*.json ./
RUN npm install
COPY . .
```

### 2. Unnecessary Files
Create a `.dockerignore` file:
```
node_modules
npm-debug.log
.git
.gitignore
.env
*.md
```

### 3. Using Latest Tag
```dockerfile
# Bad - might break unexpectedly
FROM node:latest

# Good - explicit version
FROM node:14.17.0
```

## Building and Testing

### Build Command
```bash
docker build -t myapp:1.0 .
```

### Test the Image
```bash
docker run -d -p 3000:3000 --name myapp myapp:1.0
curl http://localhost:3000
```

## Next Steps
- Explore Docker Compose
- Learn about multi-architecture builds
- Understand image signing and verification
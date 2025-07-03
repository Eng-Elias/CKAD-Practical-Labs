# 2.2 Tagging Container Images

## Understanding Image Tags

Tags are a crucial component of container images that help in versioning and managing different variants of the same image. They serve as human-readable identifiers for specific versions or configurations of an image.

## Basic Tag Management

### Listing Images
```bash
docker image ls
```
Shows all locally stored images along with their tags and sizes.

### Image History and Layers
```bash
# Modern syntax (preferred)
docker image history [IMAGE]

# Legacy syntax
docker history [IMAGE]
```
Displays the layers that make up an image, showing the commands that created each layer.

### Removing Images
```bash
docker image rm [IMAGE]
```
Removes one or more images from local storage.

## Working with Tags

### Default Tag Behavior
- If no tag is specified, `latest` is used by default
- Example: `nginx` is equivalent to `nginx:latest`

### Tagging Images
```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### Common Tagging Patterns
1. **Version Tags**: `myapp:1.0.0`
2. **Environment Tags**: `myapp:production`, `myapp:staging`
3. **Feature Tags**: `myapp:feature-x`
4. **Build Number Tags**: `myapp:build-123`

### Tagging for Registries
```bash
# Tag for local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Tag for Docker Hub
docker tag myapp:1.0 username/myapp:1.0
```

## Best Practices for Tagging

### 1. Always Use Explicit Tags
- Avoid relying on the `latest` tag in production
- Be specific about versions to ensure consistency

### 2. Tag for Multiple Environments
```bash
docker tag myapp:1.0 myapp:production
docker tag myapp:1.0 myapp:staging
```

### 3. Use Semantic Versioning
- `MAJOR.MINOR.PATCH` format (e.g., 1.2.3)
- Follow semantic versioning principles

### 4. Clean Up Old Tags
Regularly remove unused tags to save disk space:
```bash
docker image prune -a
```

## Practical Example: Working with a Local Registry

1. Start a local registry:
   ```bash
   docker run -d -p 5000:5000 --name registry registry:2
   ```

2. Pull and tag an image:
   ```bash
   docker pull nginx:alpine
   docker tag nginx:alpine localhost:5000/nginx:alpine
   ```

3. Push to the local registry:
   ```bash
   docker push localhost:5000/nginx:alpine
   ```

4. Verify the push:
   ```bash
   curl http://localhost:5000/v2/_catalog
   ```

## Common Issues and Solutions

### Untagged Images (Dangling Images)
```bash
# List dangling images
docker images -f "dangling=true"

# Remove dangling images
docker image prune
```

### Tag Conflicts
If you get a conflict when tagging:
1. Check for existing tags: `docker images | grep [image-name]`
2. Remove conflicting tags if needed: `docker rmi [image:tag]`

## Summary of Commands

| Command | Description |
|---------|-------------|
| `docker image ls` | List all images |
| `docker image history` | Show image layers |
| `docker image rm` | Remove an image |
| `docker tag` | Create a tag for an image |
| `docker push` | Push an image to a registry |
| `docker pull` | Pull an image from a registry |

## Next Steps
- Learn about different image creation methods
- Understand image layers and optimization
- Explore multi-stage builds for smaller images
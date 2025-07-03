# 1.5 Managing Container Images

## Listing Container Images

### Basic Commands
```bash
# List all local images
docker images
# or using the newer syntax
docker image ls
```

### Understanding the Output
- **REPOSITORY**: The name of the image
- **TAG**: The version of the image (defaults to 'latest' if not specified)
- **IMAGE ID**: Unique identifier for the image
- **CREATED**: When the image was created
- **SIZE**: The size of the image

## Inspecting Images

### Viewing Image Details
```bash
# Show detailed information about an image
docker image inspect <image_name_or_id>

# View specific information using JSON path
docker image inspect -f '{{.Config.Cmd}}' <image_name>
```

### Understanding Image Configuration
- **Cmd**: Default command that runs when the container starts
- **Entrypoint**: The executable that runs when the container starts
- **Env**: Environment variables
- **WorkingDir**: The working directory inside the container

## Managing Images

### Removing Images
```bash
# Remove a specific image
docker image rm <image_id_or_name>
# or the older syntax
docker rmi <image_id_or_name>

# Force remove an image (even if in use)
docker image rm -f <image_id_or_name>

# Remove all unused images
docker image prune -a
```

### Common Issues and Solutions

#### Image in Use Error
If you get an error that an image is in use:
1. List all containers (including stopped ones):
   ```bash
   docker ps -a
   ```
2. Remove the container using the image:
   ```bash
   docker rm <container_id>
   ```
3. Then remove the image

## Working with Image Tags

### Tagging Images
```bash
# Tag an existing image
docker tag <source_image> <new_image_name>:<tag>

# Example
docker tag nginx my-nginx:v1
```

### Removing Tags
```bash
# Remove a specific tag
docker rmi <repository>:<tag>
```

## Best Practices

1. **Use Specific Tags**
   - Avoid using the 'latest' tag in production
   - Always specify version numbers for better control

2. **Clean Up**
   - Regularly remove unused images to save disk space
   - Use `docker system prune` to clean up unused containers, networks, and images

3. **Inspect Before Use**
   - Always inspect third-party images before running them
   - Check the Dockerfile or source if possible

## Docker Command Structure

### New vs Old Syntax
| Old Syntax | New Syntax | Description |
|------------|------------|-------------|
| `docker images` | `docker image ls` | List images |
| `docker rmi` | `docker image rm` | Remove image |
| `docker inspect` | `docker image inspect` | Inspect image |

### Important Notes
- Commands with the `docker image` prefix are part of the newer syntax
- The older syntax is still supported but may be deprecated in the future
- The new syntax provides better organization and consistency

## Container Image Architecture

### Understanding Layers
- Container images are made up of read-only layers
- Each layer represents an instruction in the image's Dockerfile
- Layers are cached to optimize build times and reduce disk usage

### Viewing Image Layers
```bash
# Show the history of an image
docker history <image_name>

# Or using the newer syntax
docker image history <image_name>
```

This organized structure provides a comprehensive reference for managing Docker container images, covering common commands, best practices, and troubleshooting tips.
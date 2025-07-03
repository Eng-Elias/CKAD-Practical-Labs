# 2.1 Understanding Container Image Architecture

## What is a Container Image?

At its core, a container image is a tarball (`.tar` file) with additional metadata. This simple concept is the foundation of container technology, enabling efficient packaging and distribution of applications and their dependencies.

## The Layered Filesystem

### Key Concept: Image Layers
Container images are built using a **layered filesystem** approach. Each layer represents a set of file changes (additions, modifications, deletions) that are stacked on top of each other to create the final container filesystem.

### Why Layers Matter
1. **Efficiency**: Common layers can be shared between images
2. **Caching**: Unchanged layers can be reused during builds
3. **Smaller Transfers**: Only changed layers need to be downloaded

## Base Images

### What Are They?
Base images provide the foundational layer for container images. They typically contain a minimal operating system and essential system libraries.

### Common Base Images

| Base Image | Use Case | Size |
|------------|----------|------|
| `scratch` | Minimal base (empty) | 0B |
| `alpine` | Minimal Linux | ~5MB |
| `busybox` | Minimal Unix utilities | ~5MB |
| `ubuntu` | Full Ubuntu base | ~70MB |
| `fedora` | Fedora Linux | ~100MB |
| `ubi` (Red Hat) | Enterprise Linux base | ~100MB |

### Choosing a Base Image
- **Minimal images** (Alpine, Busybox) are great for:
  - Small attack surface
  - Fast downloads
  - Reduced storage needs
- **Full OS images** (Ubuntu, Fedora) are better when:
  - You need more tools/libraries
  - Debugging is required
  - Application has many dependencies

## Image Composition

### Typical Image Structure
```
+---------------------+
|     Your App        |  ← Application layer
+---------------------+
|  App Dependencies   |  ← Runtime libraries
+---------------------+
|   Database Server   |  ← Database layer
+---------------------+
|  Base OS (e.g.,     |  ← Base image
|  Alpine/Ubuntu)     |
+---------------------+
|  Container Runtime  |  ← Managed by Docker/containerd
+---------------------+
```

### Real-world Example: Web Application
1. Base OS layer (e.g., Alpine Linux)
2. System dependencies (libc, ssl, etc.)
3. Language runtime (Node.js, Python, etc.)
4. Application dependencies (npm/pip packages)
5. Application code
6. Configuration

## How Layers Work in Practice

### Building Images
When you build a Docker image:
1. Each instruction in the Dockerfile creates a new layer
2. Layers are cached for faster subsequent builds
3. Only changed layers need to be rebuilt

### Pulling Images
```bash
docker pull nginx:latest
```
This command:
1. Downloads only the layers not already present locally
2. Verifies layer checksums
3. Assembles the layers into a single filesystem

### Viewing Image Layers
```bash
# Show the layers in an image
docker image history nginx

# Detailed layer information
docker image inspect nginx
```

## Best Practices for Image Architecture

### 1. Minimize Layers
- Combine related commands in a single `RUN` instruction
- Clean up in the same layer where you install packages

### 2. Use Multi-stage Builds
- Separate build environment from runtime environment
- Only copy necessary artifacts to the final image

### 3. Be Mindful of Layer Order
- Place frequently changing layers (like application code) at the end
- Place stable dependencies (like OS packages) at the beginning

### 4. Use .dockerignore
- Exclude unnecessary files from the build context
- Reduces build time and final image size

## Common Pitfalls

### 1. The "Latest" Tag Trap
- Avoid using `:latest` in production
- Always use specific version tags for reproducibility

### 2. Unnecessary Files in Layers
- Temporary files should be cleaned in the same layer they're created
- Use multi-stage builds to exclude build tools from final images

### 3. Large Base Images
- Be aware of what's included in your base image
- Consider using minimal base images when possible

## Advanced Topics

### Overlay Filesystems
- Modern container runtimes use overlay filesystems (like overlay2)
- These enable efficient layer stacking and copy-on-write semantics

### Image Signing and Verification
- Sign images to ensure authenticity
- Verify signatures before pulling images

### Image Scanning
- Scan images for vulnerabilities
- Integrate scanning into your CI/CD pipeline

## Practical Exercise

1. Pull an image and examine its layers:
   ```bash
   docker pull nginx:alpine
   docker image history nginx:alpine
   ```

2. Build a multi-stage Dockerfile and compare image sizes

3. Experiment with different base images and compare their sizes and contents

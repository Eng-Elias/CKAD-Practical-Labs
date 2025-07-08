# CKAD Practice Question 11: Working with Container Images (Docker & Podman)

## Question
You are tasked with building and managing container images using both Docker and Podman on instance `ckad9043`. The container runs a Go application that outputs information to STDOUT.

**Tasks**:
1.  **Change the Dockerfile**:
    -   The files to build the image are located at `/opt/course/11/image`.
    -   The value of the environment variable `SUN_CIPHER_ID` should be set to the hardcoded value `5b9c1065-e39d-4a43-a04a-e59bcea3e03f`.

2.  **Build and Push with Docker**:
    -   Build the image using Docker, named `registry.killer.sh:5000/sun-cipher`.
    -   Tag the image as `latest` and `v1-docker`.
    -   Push both tags to the registry.

3.  **Build and Push with Podman**:
    -   Build the image using Podman, named `registry.killer.sh:5000/sun-cipher`.
    -   Tag the image as `v1-podman`.
    -   Push the tag to the registry.

4.  **Run a Container with Podman**:
    -   Run a container using Podman, which keeps running in the background.
    -   Name the container `sun-cipher`.
    -   Use the image `registry.killer.sh:5000/sun-cipher:v1-podman`.

5.  **Collect Logs and Container Info**:
    -   Write the logs your container `sun-cipher` produced into `/opt/course/11/logs`.
    -   Write a list of all running Podman containers into `/opt/course/11/containers`.

**Important Notes**:
-   Run all commands as user `candidate`.
-   For Docker, use `sudo docker`.

## Solution

### Step 1: Modify the Dockerfile
```bash
# Connect to the instance
ssh ckad9043

# Navigate to the directory containing the Dockerfile
cd /opt/course/11/image

# Check the current Dockerfile content
cat Dockerfile

# Edit the Dockerfile to set the environment variable
# Find the line with SUN_CIPHER_ID and update it to:
# ENV SUN_CIPHER_ID=5b9c1065-e39d-4a43-a04a-e59bcea3e03f
# You can use sed or a text editor like vi
sed -i 's/SUN_CIPHER_ID=.*/SUN_CIPHER_ID=5b9c1065-e39d-4a43-a04a-e59bcea3e03f/' Dockerfile
```

### Step 2: Build and Push with Docker
```bash
# Build the Docker image with both tags from within /opt/course/11/image
sudo docker build -t registry.killer.sh:5000/sun-cipher:latest \
                 -t registry.killer.sh:5000/sun-cipher:v1-docker .

# Push both images to the registry
sudo docker push registry.killer.sh:5000/sun-cipher:latest
sudo docker push registry.killer.sh:5000/sun-cipher:v1-docker
```

### Step 3: Build and Push with Podman
```bash
# Build the image using Podman from within /opt/course/11/image
podman build -t registry.killer.sh:5000/sun-cipher:v1-podman .

# Push the Podman image to the registry
podman push registry.killer.sh:5000/sun-cipher:v1-podman
```

### Step 4: Run a Container with Podman
```bash
# Run the container in detached mode with the specified name
podman run -d --name sun-cipher registry.killer.sh:5000/sun-cipher:v1-podman

# Verify the container is running
podman ps
```

### Step 5: Collect Logs and Container Information
```bash
# Save container logs to the specified file
podman logs sun-cipher > /opt/course/11/logs

# Save list of running Podman containers to the specified file
podman ps > /opt/course/11/containers

# Verify the files have been created
ls -l /opt/course/11/
cat /opt/course/11/logs
cat /opt/course/11/containers
```

## Explanation

### Key Concepts
1.  **Docker and Podman**: Both are container runtimes. Podman is daemonless and can be run as a non-root user without `sudo`, whereas Docker typically requires `sudo` due to its client-server architecture.
2.  **Dockerfile ENV**: The `ENV` instruction sets a persistent environment variable in the image.
3.  **Image Tagging**: Applying multiple tags (`-t`) allows an image to be referenced by different names, which is useful for versioning (e.g., `v1-docker`) and environment promotion (e.g., `latest`).
4.  **Container Registry**: A storage and distribution system for container images. Pushing an image makes it available to other systems.
5.  **Detached Mode**: The `-d` flag runs a container in the background, freeing up the terminal.

### Why This Solution Works
-   The `Dockerfile` is correctly modified to include the required environment variable.
-   `sudo docker build` and `podman build` are used to create the images from the same source but with different tools and tags.
-   `docker push` and `podman push` upload the images to the shared registry.
-   `podman run -d` starts the container in the background as required.
-   Standard output redirection (`>`) is used to save the command output (`podman logs` and `podman ps`) to the specified files.

### Exam Tips
1.  **Tool Differences**: Be mindful of when to use `sudo` (for Docker) and when not to (for Podman, unless root privileges are needed for other reasons).
2.  **Fully Qualified Image Names**: When pushing or pulling from a registry, always use the full image name, including the registry URL and port (e.g., `registry.killer.sh:5000/...`).
3.  **Context is Key**: The `.` at the end of the `build` command is crucial; it tells the build engine to use the current directory as the build context.
4.  **Verification is Crucial**: After each major step (build, push, run), use verification commands (`docker images`, `podman ps`, `ls`, `cat`) to ensure the step was successful.

### Common Mistakes to Avoid
-   Forgetting to use `sudo` with Docker commands.
-   Not including the registry port (`:5000`) in the image name.
-   Running the `build` command from the wrong directory (not the one with the Dockerfile).
-   Forgetting to push the images to the registry after building them.
-   Using incorrect file paths when saving logs and container information.

## Related Commands
```bash
# List all Docker images
sudo docker images

# List all Podman images
podman images

# View running containers (Docker and Podman)
sudo docker ps
podman ps

# Stop and remove a Podman container
podman stop sun-cipher
podman rm sun-cipher

# Remove a Podman image
podman rmi registry.killer.sh:5000/sun-cipher:v1-podman

# View build history of an image
podman history registry.killer.sh:5000/sun-cipher:v1-podman
```

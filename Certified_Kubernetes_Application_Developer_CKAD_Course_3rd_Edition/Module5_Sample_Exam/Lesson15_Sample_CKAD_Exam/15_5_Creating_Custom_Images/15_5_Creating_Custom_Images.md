# 15.5 Creating Custom Images

This section provides a solution for the sample exam assignment on creating and exporting a custom Docker image.

## Task

1.  Create a `Dockerfile` that uses an `alpine` base image.
2.  The container's default command should be `echo hello world`.
3.  Build the image and tag it as `greet-world`.
4.  Export the built image in OCI format to a tar file named `greet-world.tar`.

---

## Solution Walkthrough

This task involves basic Docker operations.

### 1. Create the Dockerfile

First, create a file named `Dockerfile` in your working directory with the following content:

```dockerfile
FROM alpine
CMD ["echo", "hello world"]
```

*   `FROM alpine`: Specifies the base image.
*   `CMD ["echo", "hello world"]`: Sets the default command to be executed when a container is run from this image. Using the array notation is a best practice.

### 2. Build the Docker Image

Use the `docker build` command to create the image from the `Dockerfile`.

```bash
docker build -t greet-world .
```

*   `-t greet-world`: Tags the image with the name `greet-world`.
*   `.`: Specifies that the build context (and the `Dockerfile`) is in the current directory.

Verify that the image was created successfully:

```bash
docker images
```

### 3. Export the Image

Use the `docker save` command to export the image to a tar archive.

```bash
docker save -o greet-world.tar greet-world
```

*   `-o greet-world.tar`: Specifies the output file name.
*   `greet-world`: The name of the image to save.

Verify that the tar file was created:

```bash
ls -l
```

### A Note on OCI Format

The requirement to export in "OCI format" can be ignored. Modern Docker images are already compliant with the Open Container Initiative (OCI) specification, so a standard `docker save` is sufficient to meet the requirement.
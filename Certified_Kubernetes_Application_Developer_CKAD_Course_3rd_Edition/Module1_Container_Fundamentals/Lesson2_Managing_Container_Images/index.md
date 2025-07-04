# Lesson 2: Managing Container Images

## Overview
This lesson dives deep into container images, covering their architecture, creation, and management. You'll learn how to build, tag, and manage container images effectively, which is crucial for working with Kubernetes.

## Learning Objectives
- Understand container image architecture and layers
- Learn to tag and version container images
- Explore different image creation methods
- Build custom container images using Dockerfile/Containerfile
- Understand image management best practices
- Work with image registries

## Lessons

### [2.1 Understanding Image Architecture](2_1_Understanding_Image_Architecture/2_1_Understanding_Image_Architecture.md)
- Container image structure
- Layered filesystem concept
- Base images and their importance
- Image layers and caching mechanisms

### [2.2 Tagging Container Images](2_2_Tagging_Container_Images/2_2_Tagging_Container_Images.md)
- Image naming conventions
- Tagging strategies
- Versioning best practices
- Working with multiple tags

### [2.3 Understanding Image Creation Options](2_3_Understanding_Image_Creation_Options/2_3_Understanding_Image_Creation_Options.md)
- Different approaches to create container images
- Manual vs. automated image creation
- Choosing the right base image
- Security considerations in image creation

### [2.4 Building Custom Container Images](2_4_Building_Custom_Container_Images/2_4_Building_Custom_Container_Images.md)
- Planning your container image
- Best practices for building efficient images
- Multi-stage builds
- Optimizing image size and security

### [2.5 Using Dockerfile/Containerfile](2_5_Using_Dockerfile_Containerfile/2_5_Using_Dockerfile_Containerfile.md)
- Dockerfile/Containerfile syntax and instructions
- Common Dockerfile patterns
- Best practices for maintainable Dockerfiles
- Building and testing container images

### [2.6 Creating Images with Docker Commit](2_6_Creating_Images_with_Docker_Commit/2_6_Creating_Images_with_Docker_Commit.md)
- When to use docker commit
- Interactive container modification
- Limitations and considerations
- Use cases for manual image creation

## Hands-on Exercises
- Build custom container images
- Implement multi-stage builds
- Optimize image layers
- Work with image tags and versions
- Push and pull images from registries

## Best Practices
- Keep images small and focused
- Use specific version tags
- Implement proper layer caching
- Scan images for vulnerabilities
- Sign and verify images

## Next Steps
After completing this lesson, you'll have a solid understanding of container images and be ready to explore more advanced container orchestration concepts in the next module [Lesson 3: Understanding Kubernetes](../../Module2_Kubernetes_Essentials/index.md).
# Lesson 3: Understanding Kubernetes

## Overview
This lesson provides a comprehensive introduction to Kubernetes, covering its core functions, architecture, and ecosystem. You'll learn about Kubernetes origins, its management interfaces, and how it integrates with cloud providers.

## Learning Objectives
- Understand the core functions and benefits of Kubernetes
- Learn about Kubernetes architecture and components
- Explore Kubernetes management interfaces
- Understand essential API resources
- Learn about Kubernetes deployment options

## Lessons

### [3.1 Understanding Kubernetes Core Functions](3_1_Understanding_Kubernetes_Core_Functions/3_1_Understanding_Kubernetes_Core_Functions.md)
- Introduction to Kubernetes and CNCF
- Core functionality and features
- Cloud-native computing concepts
- Kubernetes as a container orchestrator

### [3.2 Understanding Kubernetes Origins](3_2_Understanding_Kubernetes_Origins/3_2_Understanding_Kubernetes_Origins.md)
- History and development of Kubernetes
- Relationship with Google's Borg system
- Open-source evolution and community
- Kubernetes release cycle and versioning

### [3.3 Using Kubernetes in Google Cloud](3_3_Using_Kubernetes_in_Google_Cloud/3_3_Using_Kubernetes_in_Google_Cloud.md)
- Google Kubernetes Engine (GKE) overview
- Creating and managing clusters in GCP
- Integration with Google Cloud services
- Best practices for GKE

### [3.4 Understanding Kubernetes Management Interfaces](3_4_Understanding_Kubernetes_Management_Interfaces/3_4_Understanding_Kubernetes_Management_Interfaces.md)
- kubectl command-line tool
- Kubernetes Dashboard
- Declarative vs imperative management
- Working with YAML manifests

### [3.5 Understanding Kubernetes Architecture](3_5_Understanding_Kubernetes_Architecture/3_5_Understanding_Kubernetes_Architecture.md)
- Control plane components
- Node components
- Cluster communication
- High availability concepts

### [3.6 Exploring Essential API Resources](3_6_Exploring_Essential_API_Resources/3_6_Exploring_Essential_API_Resources.md)
- Kubernetes API basics
- Core API resources
- Custom Resource Definitions (CRDs)
- API versioning and deprecation

## Hands-on Exercises
- Explore Kubernetes cluster components
- Use kubectl to interact with the cluster
- Create and manage basic resources
- Inspect cluster state and resources

## Key Concepts
- **Container Orchestration**: Managing containerized applications across multiple hosts
- **Control Plane**: The brain of Kubernetes that manages the cluster
- **Nodes**: Worker machines that run containerized applications
- **Pods**: The smallest deployable units in Kubernetes
- **Controllers**: Ensure the desired state matches the actual state

## Best Practices
- Understand the Kubernetes architecture before deployment
- Use declarative configuration for all resources
- Follow the principle of least privilege
- Regularly update to supported Kubernetes versions
- Monitor cluster health and performance

## Next Steps
After completing this lesson, you'll be ready to set up your own Kubernetes environment in [Lesson 4: Creating a Lab Environment](../Lesson4_Creating_a_Lab_Environment/index.md).
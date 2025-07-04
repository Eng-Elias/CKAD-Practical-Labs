# Certified Kubernetes Application Developer (CKAD) Course - 3rd Edition

## Introduction

Since its first release on July 21, 2015, Kubernetes has revolutionized the way IT infrastructure is managed. It offers a scalable way to run containers in public cloud, on-premise data centers, or even as a test environment on your own laptop.

### About Kubernetes Certifications

- **KCNA (Kubernetes and Cloud Native Associate)**: The entry-level certification
- **CKAD (Certified Kubernetes Application Developer)**: Focuses on application deployment and management (this course)
- **CKA (Certified Kubernetes Administrator)**: Covers cluster management and operations
- **CKS (Certified Kubernetes Security Specialist)**: Specialized in Kubernetes security

### About This Course

This course is a preparation for the CKAD exam, but it's also valuable for anyone who wants to master Kubernetes application deployment. You'll learn essential skills for running applications in Kubernetes, including:

- Container fundamentals and Kubernetes basics
- Pod management and configuration
- Application deployment and scaling
- Networking, storage, and security
- Advanced CKAD tasks and troubleshooting

Even if you're not planning to get certified, these skills are crucial for anyone working with Kubernetes in development or operations roles.

### Course Instructor

Sander van Vugt is a specialist in open source platforms and infrastructure, with expertise in Linux and Kubernetes. He is also the founder of the Living Open Source Foundation, a non-profit organization that provides open source education in various African countries.

## Course Overview
This directory contains theoretical concepts and practical exercises for the Certified Kubernetes Application Developer (CKAD) certification preparation course. The course is designed to master the skills needed to design, build, configure, and expose cloud-native applications for Kubernetes.

## Getting Started

### Prerequisites
- Basic understanding of Linux command line
- Familiarity with container concepts (Docker, containers, images)
- A working Kubernetes environment (Minikube, Docker Desktop, or cloud-based cluster)

### Course Structure
The course is divided into modules, each focusing on specific aspects of Kubernetes application development. Each module contains multiple lessons with detailed explanations and hands-on exercises.

## Course Modules

### [Module 1: Container Fundamentals](./Module1_Container_Fundamentals/index.md)
- [Lesson 1: Understanding and Using Containers](./Module1_Container_Fundamentals/Lesson1_Understanding_and_Using_Containers/index.md)
  - Container concepts and architecture
  - Container lifecycle management
  - Working with container registries
- [Lesson 2: Managing Container Images](./Module1_Container_Fundamentals/Lesson2_Managing_Container_Images/index.md)
  - Image architecture and layers
  - Building and optimizing images
  - Image versioning and best practices

### [Module 2: Kubernetes Essentials](./Module2_Kubernetes_Essentials/index.md)
- [Lesson 3: Understanding Kubernetes](./Module2_Kubernetes_Essentials/Lesson3_Understanding_Kubernetes/index.md)
  - Kubernetes architecture and components
  - Management interfaces and API resources
- [Lesson 4: Creating a Lab Environment](./Module2_Kubernetes_Essentials/Lesson4_Creating_a_Lab_Environment/index.md)
  - Setting up Minikube
  - Running your first application
- [Lesson 5-7: Pod Management](./Module2_Kubernetes_Essentials/Lesson5_Managing_Pods_Basic_Features/index.md)
  - Basic and advanced Pod features
  - Deployments and scaling
- [Lesson 8: Managing Networking](./Module2_Kubernetes_Essentials/Lesson8_Managing_Networking/index.md)
  - Network policies and security
  - Service discovery and load balancing
- [Lesson 9: Managing Ingress](./Module2_Kubernetes_Essentials/Lesson9_Managing_Ingress/index.md)
  - Ingress controller and rules
  - TLS termination and authentication
- [Lesson 10: Managing Kubernetes Storage](./Module2_Kubernetes_Essentials/Lesson10_Managing_Kubernetes_Storage/index.md)
  - Persistent Volumes and Claims
  - StorageClasses
- [Lesson 11: Managing ConfigMaps and Secrets](./Module2_Kubernetes_Essentials/Lesson11_Managing_ConfigMaps_and_Secrets/index.md)
  - Configuration management
  - Sensitive data handling

### [Module 3: Building and Exposing Scalable Applications](./Module3_Building_and_Exposing_Scalable_Applications/index.md)
- [Lesson 12: Using the API](./Module3_Building_and_Exposing_Scalable_Applications/Lesson12_Using_the_API/index.md)
  - Kubernetes API fundamentals
  - Authentication and authorization
  - RBAC and ServiceAccounts
- [Lesson 13: Deploying Application the DevOps Way](./Module3_Building_and_Exposing_Scalable_Applications/Lesson13_Deploying_Application_the_DevOps_Way/index.md)
  - Helm package manager
  - Working with Helm charts
  - Using Kustomize

### [Module 4: Advanced CKAD Tasks](./Module4_Advanced_CKAD_Tasks/index.md)
- [Lesson 13: Advanced Deployment Strategies](./Module4_Advanced_CKAD_Tasks/Lesson13_Deploying_Application_the_DevOps_Way/index.md)
  - Blue-Green deployments
  - Canary deployments
  - Custom Resource Definitions (CRDs)
  - Kubernetes Operators
  - StatefulSets
- [Lesson 14: Troubleshooting Kubernetes](./Module4_Advanced_CKAD_Tasks/Lesson14_Troubleshooting_Kubernetes/index.md)
  - Troubleshooting methodology
  - Application and access issues
  - Cluster monitoring and events
  - Authentication problems
  - Using probes effectively

### [Module 5: Sample CKAD Exam](./Module5_Sample_Exam/index.md)
- [Lesson 15: Sample CKAD Exam](./Module5_Sample_Exam/Lesson15_Sample_CKAD_Exam/index.md)
  - Exam tips and strategies
  - Practice exercises covering all exam objectives
  - Real-world scenarios
  - Time management techniques
  - Common pitfalls to avoid

## Prerequisites
- Basic understanding of container concepts
- Familiarity with Linux command line
- Docker fundamentals (recommended)

## Getting Started
1. Set up your lab environment with Minikube or a Kubernetes cluster
2. Install `kubectl` command-line tool

## Resources
- [Certified KUBERNETES Application Developer Full Course](https://youtu.be/4rxIiOmKmiE?si=IdXn9GZbBeZY-f6q)
- [Course Respurces](https://github.com/sandervanvugt/ckad)
- [Official CNCF CKAD Curriculum](https://github.com/cncf/curriculum)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## License
This material is available for educational purposes only.

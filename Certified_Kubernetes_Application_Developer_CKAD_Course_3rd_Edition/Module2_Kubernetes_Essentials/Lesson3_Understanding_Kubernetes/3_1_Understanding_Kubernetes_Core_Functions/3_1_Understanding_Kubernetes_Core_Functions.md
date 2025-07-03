# 3.1 Understanding Kubernetes Core Functions

## What is Kubernetes?

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. The name comes from the Greek word meaning "captain of the ship," which is why many Kubernetes ecosystem symbols reference shipping and containers.

## Kubernetes and CNCF

- Currently owned by the Cloud Native Computing Foundation (CNCF)
- CNCF is an open-source foundation within the Linux Foundation
- Ensures Kubernetes remains open and available to everyone
- Kubernetes is not just software or a product, but an entire ecosystem

## CKAD Focus

- CKAD focuses on the core Kubernetes solution, not third-party add-ons
- The certification is vendor-neutral regarding ecosystem components
- Core solution includes essential functionality without vendor-specific implementations

## Cloud-Native Computing

Kubernetes implements a platform for container-based applications in a cloud-native environment:
- Applications don't have a direct relation to any specific server
- All application requirements are stored and managed in the cloud
- Provides specific properties needed for cloud-native applications
- Offers Kubernetes resources through defined APIs

## Kubernetes Distributions

### Vanilla Kubernetes
- Directly from source code hosted by CNCF
- Many organizations use this basic version

### Custom Distributions
- Add specific functionality and ecosystem solutions
- Examples:
  - Google Anthos
  - Red Hat OpenShift
  - SUSE Rancher
  - Canonical Kubernetes

## Release Cycle

- New releases every three months
- New API versions may be introduced
- Old features may be deprecated
- Deprecated features are removed within two releases
- Course materials are regularly updated to reflect changes

## Why Kubernetes?

- Open source and available to all
- Strong common codebase
- Companies can focus on adding value rather than reinventing the wheel
- No significant competition in the container orchestration space

> **Note:** Docker Swarm, while once a competitor, has become largely insignificant compared to Kubernetes.

## Key Takeaways

- Kubernetes is the industry standard for container orchestration
- It provides the necessary tools to run applications in a cloud-native way
- The ecosystem is built around a strong, open-source core
- Regular updates ensure the platform stays current with industry needs

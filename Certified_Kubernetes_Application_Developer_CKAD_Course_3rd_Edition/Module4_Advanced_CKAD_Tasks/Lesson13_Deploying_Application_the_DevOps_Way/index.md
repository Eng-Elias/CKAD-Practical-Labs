# Module 3: Building and Exposing Scalable Applications

## Overview
This module focuses on building and exposing applications in Kubernetes, covering service networking, application access, and advanced deployment strategies. You'll learn how to make your applications accessible, scalable, and maintainable in a production environment.

## Prerequisites
- Completion of [Module 2: Kubernetes Essentials](../../Module2_Kubernetes_Essentials/index.md)
- Working knowledge of Pods, Deployments, and Services
- Basic understanding of networking concepts

## Learning Path

### [Lesson 12: Managing Application Access](Lesson12_Managing_Application_Access/index.md)
- Understanding Service resources
- Service networking concepts
- ClusterIP and NodePort services
- Configuring Ingress resources
- Ingress controllers setup

### [Lesson 13: Deploying Applications the DevOps Way](Lesson13_Deploying_Application_the_DevOps_Way/index.md)
- Using Helm package manager
- Working with Helm charts
- Kustomize for configuration management
- Blue-Green deployments
- Canary deployments
- StatefulSets for stateful applications

### [Lesson 14: Using the Kubernetes API](Lesson12_Using_the_API/index.md)
- Understanding the Kubernetes API
- API authentication and authorization
- Working with ServiceAccounts
- Role-Based Access Control (RBAC)
- Custom Resource Definitions (CRDs)
- Operators and custom controllers

## Key Concepts
- **Service Discovery**: How applications find each other
- **Load Balancing**: Distributing traffic across Pods
- **Ingress**: Managing external access to services
- **CI/CD**: Continuous Integration and Deployment
- **GitOps**: Infrastructure as Code
- **Custom Resources**: Extending the Kubernetes API

## Hands-on Exercises
1. Expose an application using different Service types
2. Configure an Ingress controller and resources
3. Package an application using Helm
4. Implement a blue-green deployment
5. Create and manage Custom Resource Definitions
6. Secure API access using RBAC

## Best Practices
- Use Services for internal communication
- Implement proper network policies
- Use Ingress for HTTP/HTTPS routing
- Version your Helm charts
- Follow the principle of least privilege for API access
- Monitor API server metrics

## Resources
- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)

## Next Steps
After completing this module, you'll be ready to tackle advanced Kubernetes topics in [Module 4: Advanced CKAD Tasks](../../Module4_Advanced_CKAD_Tasks/index.md), where you'll learn about troubleshooting, monitoring, and optimizing Kubernetes applications.
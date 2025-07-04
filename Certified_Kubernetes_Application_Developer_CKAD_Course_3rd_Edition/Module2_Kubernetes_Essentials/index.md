# Module 2: Kubernetes Essentials

## Overview
This module dives into the core concepts of Kubernetes, providing hands-on experience with essential Kubernetes resources and features. Building on the container fundamentals from Module 1, you'll learn how to deploy, manage, and scale applications in a Kubernetes cluster.

## Prerequisites
- Completion of [Module 1: Container Fundamentals](../Module1_Container_Fundamentals/index.md)
- Basic understanding of container concepts
- Access to a Kubernetes cluster (Minikube or cloud-based)

## Learning Path

### [Lesson 3: Understanding Kubernetes](Lesson3_Understanding_Kubernetes/index.md)
- Kubernetes core functions and architecture
- Management interfaces and API resources
- Cloud integration and deployment options

### [Lesson 4: Creating a Lab Environment](Lesson4_Creating_a_Lab_Environment/index.md)
- Setting up Minikube on different platforms
- Verifying cluster functionality
- Running your first application

### [Lesson 5: Managing Pods - Basic Features](Lesson5_Managing_Pods_Basic_Features/index.md)
- Understanding pods and YAML
- Multi-container pods
- Working with init containers and namespaces

### [Lesson 6: Managing Pods - Advanced Features](Lesson6_Managing_Pod_Advanced_Features/index.md)
- Troubleshooting pods and applications
- Security contexts and resource management
- Jobs and CronJobs

### [Lesson 7: Managing Deployments](Lesson7_Managing_Deployments/index.md)
- Deployment strategies
- Scaling applications
- Rolling updates and rollbacks

### [Lesson 8: Managing Networking](Lesson8_Managing_Networking/index.md)
- Kubernetes networking model and CNI
- Service types: ClusterIP, NodePort, LoadBalancer
- DNS and service discovery
- Network policies for security
- Troubleshooting network issues

### [Lesson 9: Managing Ingress](Lesson9_Managing_Ingress/index.md)
- Ingress concepts and architecture
- Configuring Ingress controllers
- Path-based and host-based routing
- TLS termination and security
- Troubleshooting Ingress resources

### [Lesson 10: Managing Kubernetes Storage](Lesson10_Managing_Kubernetes_Storage/index.md)
- Persistent storage concepts
- PersistentVolumes and PersistentVolumeClaims
- StorageClass for dynamic provisioning
- Volume types and use cases
- Stateful application patterns

### [Lesson 11: Managing ConfigMaps and Secrets](Lesson11_Managing_ConfigMaps_and_Secrets/index.md)
- Decoupling configuration from applications
- Creating and using ConfigMaps
- Managing sensitive data with Secrets
- Security best practices
- Docker registry authentication

## Key Concepts
- **Kubernetes Architecture**: Master and worker nodes, control plane components, and their interactions
- **Pods and Workloads**: The smallest deployable units and their management
- **Services and Networking**: ClusterIP, NodePort, LoadBalancer, and Ingress
- **Storage Management**: PersistentVolumes, PersistentVolumeClaims, and StorageClasses
- **Configuration and Secrets**: Managing application configuration and sensitive data
- **Security**: NetworkPolicies, RBAC, and security contexts

## Hands-on Labs
1. Set up a local Kubernetes environment using Minikube or kind
2. Deploy and manage containerized applications
3. Configure networking with Services and Ingress
4. Implement persistent storage solutions
5. Manage application configuration with ConfigMaps and Secrets
6. Implement security controls and policies
7. Monitor and troubleshoot cluster resources

## Best Practices
- Use declarative configuration with YAML files
- Implement proper resource requests and limits
- Use namespaces for logical separation
- Follow the principle of least privilege
- Implement proper health checks
- Monitor application and cluster metrics
- Document configurations and procedures

## Common Commands
```bash
# Get cluster information
kubectl cluster-info

# View nodes
kubectl get nodes

# Create resources from YAML
kubectl apply -f <file>.yaml

# View pod logs
kubectl logs <pod-name>

# Access a pod's shell
kubectl exec -it <pod-name> -- /bin/sh

# Port-forward to a pod
kubectl port-forward <pod-name> <local-port>:<pod-port>
```

## Resources
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)

## Next Steps
After completing this module, you'll be ready to move on to [Module 3: Building and Exposing Scalable Applications](../../Module3_Building_and_Exposing_Scalable_Applications/index.md), where you'll learn about advanced deployment strategies, Helm, and more.

## Learning Outcomes
By the end of this module, you will be able to:
- Explain Kubernetes architecture and components
- Deploy and manage applications in Kubernetes
- Configure networking and storage for applications
- Manage application configuration and secrets
- Scale and update applications with minimal downtime

## Resources
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Next Steps
After completing this module, you'll be ready to explore more advanced Kubernetes concepts in subsequent modules, including networking, security, and application deployment strategies [Module 3: Building and Exposing Scalable Applications](../Module3_Building_and_Exposing_Scalable_Applications/index.md).
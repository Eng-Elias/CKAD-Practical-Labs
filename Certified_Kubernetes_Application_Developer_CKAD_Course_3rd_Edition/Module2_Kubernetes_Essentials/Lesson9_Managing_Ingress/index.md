# Lesson 9: Managing Ingress

## Overview
This lesson focuses on Ingress, a Kubernetes resource that manages external access to services in a cluster, typically HTTP/HTTPS. You'll learn how to configure and manage Ingress resources, set up Ingress controllers, and implement various routing rules for your applications.

## Learning Objectives
- Understand Ingress concepts and use cases
- Configure and manage Ingress resources
- Set up and configure Ingress controllers
- Implement path-based and host-based routing
- Secure Ingress with TLS

## Lessons

### [9.1 Understanding Ingress](9_1_Understanding_Ingress/9_1_Understanding_Ingress.md)
- What is Ingress?
- Ingress vs. other service types
- Ingress controllers overview
- Use cases for Ingress

### [9.2 Configuring Minikube Ingress Controller](9_2_Configuring_Minikube_Ingress_Controller/9_2_Configuring_Minikube_Ingress_Controller.md)
- Enabling Ingress in Minikube
- Deploying the Nginx Ingress controller
- Verifying the Ingress controller
- Basic Ingress examples

### [9.3 Using Ingress](9_3_Using_Ingress/9_3_Using_Ingress.md)
- Creating basic Ingress resources
- Path-based routing
- Host-based routing
- Default backends

### [9.4 Configuring Ingress Rules](9_4_Configuring_Ingress_Rules/9_4_Configuring_Ingress_Rules.md)
- Path types and matching
- Multiple rules and paths
- Wildcard hosts
- Annotations and custom configurations

### [9.5 Understanding IngressClass](9_5_Understanding_IngressClass/9_5_Understanding_IngressClass.md)
- What is IngressClass?
- Default IngressClass
- Specifying IngressClass in Ingress resources
- Multiple Ingress controllers

### [9.6 Troubleshooting Ingress](9_6_Troubleshooting_Ingress/9_6_Troubleshooting_Ingress.md)
- Common Ingress issues
- Debugging Ingress controllers
- Viewing Ingress controller logs
- Network policies and Ingress

## Hands-on Exercises
1. Set up an Ingress controller in Minikube
2. Create basic Ingress resources for your applications
3. Implement path-based and host-based routing
4. Configure TLS termination
5. Troubleshoot common Ingress issues

## Best Practices
- Use descriptive hostnames and paths
- Implement proper TLS termination
- Monitor Ingress controller metrics
- Use annotations for controller-specific features
- Document routing rules and configurations

## Common Commands
```bash
# List all Ingress resources
kubectl get ingress

# Get detailed Ingress information
kubectl describe ingress <ingress-name>

# Check Ingress controller logs
kubectl logs -n <namespace> <ingress-controller-pod>

# Enable Ingress in Minikube
minikube addons enable ingress

# Check Ingress controller status
kubectl get pods -n ingress-nginx
```

## Resources
- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

## Next Steps
After completing this lesson, you'll be ready to explore [Lesson 10: Managing Kubernetes Storage](../Lesson10_Managing_Kubernetes_Storage/index.md), where you'll learn how to manage persistent storage for your applications in Kubernetes.
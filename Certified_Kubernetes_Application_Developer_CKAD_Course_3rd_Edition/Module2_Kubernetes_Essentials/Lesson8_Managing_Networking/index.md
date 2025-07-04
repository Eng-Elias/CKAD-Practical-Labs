# Lesson 8: Managing Networking

## Overview
This lesson covers the fundamental networking concepts in Kubernetes, including how pods communicate with each other and how services enable network access to applications. You'll learn about different service types, DNS, and network policies for securing pod-to-pod communication.

## Learning Objectives
- Understand Kubernetes networking model
- Work with different service types (ClusterIP, NodePort, LoadBalancer)
- Configure DNS for service discovery
- Implement NetworkPolicies for security
- Troubleshoot common networking issues

## Lessons

### [8.1 Understanding Kubernetes Networking](8_1_Understanding_Kubernetes_Networking/8_1_Understanding_Kubernetes_Networking.md)
- Pod networking model
- Container Network Interface (CNI)
- Network plugins overview
- Pod-to-Pod communication

### [8.2 Understanding Services](8_2_Understanding_Services/8_2_Understanding_Services.md)
- Service concepts and types
- Service discovery
- kube-proxy and iptables/ipvs
- Headless services

### [8.3 Creating Services](8_3_Creating_Services/8_3_Creating_Services.md)
- Creating ClusterIP services
- Configuring NodePort services
- Setting up LoadBalancer services
- Service selectors and labels

### [8.4 Using Service Resources in Microservices](8_4_Using_Service_Resources_in_Microservices/8_4_Using_Service_Resources_in_Microservices.md)
- Service-to-service communication
- Service discovery patterns
- Load balancing strategies
- Session affinity

### [8.5 Understanding Services and DNS](8_5_Understanding_Services_and_DNS/8_5_Understanding_Services_and_DNS.md)
- Kubernetes DNS service
- DNS for services and pods
- DNS policies
- Debugging DNS issues

### [8.6 Understanding and Configuring NetworkPolicy](8_6_Understanding_and_Configuring_NetworkPolicy/8_6_Understanding_and_Configuring_NetworkPolicy.md)
- NetworkPolicy concepts
- Default policies
- Ingress and egress rules
- Policy best practices

## Hands-on Exercises
1. Create and expose services using different service types
2. Configure DNS for service discovery
3. Implement NetworkPolicies to control traffic
4. Debug network connectivity issues
5. Set up a simple microservices architecture

## Best Practices
- Use descriptive labels for services
- Implement network policies following least privilege
- Monitor service endpoints
- Use readiness and liveness probes
- Document network architecture

## Common Commands
```bash
# List all services
kubectl get services

# Create a service from a YAML file
kubectl apply -f service.yaml

# Get detailed service information
kubectl describe service <service-name>

# Expose a deployment as a service
kubectl expose deployment <deployment-name> --port=80 --target-port=8080

# Apply a NetworkPolicy
kubectl apply -f network-policy.yaml
```

## Resources
- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

## Next Steps
After completing this lesson, you'll be ready to explore [Lesson 9: Managing Ingress](../Lesson9_Managing_Ingress/index.md), where you'll learn how to manage external access to services in a cluster, typically via HTTP/HTTPS.
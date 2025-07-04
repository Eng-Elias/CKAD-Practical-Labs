# 8.1 Understanding Kubernetes Networking

## Core Networking Concepts

Kubernetes networking is built around several fundamental concepts that enable communication between different components:

### Key Networking Models

1. **Pod Network**
   - Every pod gets its own IP address
   - All containers in a pod share the same network namespace
   - Pods can communicate with all other pods without NAT

2. **Node Network**
   - Physical network connecting the nodes
   - External users access the cluster through this network
   - Each node has a unique IP on this network

3. **Cluster Network**
   - Internal network that provides access to the outside
   - Nodes are connected to this network
   - Services operate at this level

## Service Types in Kubernetes

### ClusterIP (Default)
- Only accessible within the cluster
- Gets a stable internal IP address
- Used for internal service-to-service communication

### NodePort
- Exposes the service on each Node's IP at a static port
- Accessible from outside the cluster using `NodeIP:NodePort`
- Automatically creates a ClusterIP service to route traffic

### LoadBalancer
- Creates an external load balancer in cloud providers
- Automatically creates NodePort and ClusterIP services
- Routes external traffic to the service

## Networking Components

### kube-proxy
- Runs on each node
- Maintains network rules on nodes
- Enables the service abstraction

### Container Network Interface (CNI)
- Standard interface between container runtimes and network implementations
- Handles IP address management
- Manages network connectivity for containers

## Practical Example

### Basic Pod-to-Pod Communication
```yaml
# pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

---
# pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    app: backend
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 3600']
```

### Verifying Network Connectivity
```bash
# Get pod IP addresses
kubectl get pods -o wide

# Test connectivity between pods
kubectl exec -it pod-1 -- curl <pod-2-ip>
```

## Best Practices

1. **Use Services for Discovery**
   - Rely on DNS for service discovery
   - Avoid using pod IPs directly
   - Use environment variables or DNS for service lookup

2. **Network Segmentation**
   - Use NetworkPolicies to control traffic
   - Implement namespaces for logical separation
   - Consider service meshes for complex routing

3. **Performance Considerations**
   - Be aware of network plugin overhead
   - Monitor network performance
   - Consider network policies impact on performance

## Common Issues and Troubleshooting

### Pods Can't Communicate
1. Check if pods are running
2. Verify network policies
3. Check CNI plugin status

### Services Not Accessible
1. Verify service endpoints
2. Check kube-proxy status
3. Verify network policies

### DNS Resolution Issues
1. Check CoreDNS pods
2. Verify DNS configuration
3. Test DNS resolution from a pod

## Next Steps
In the next section, we'll dive deeper into Kubernetes Services and how to expose your applications using different service types.

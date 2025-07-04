# 5.1 Understanding Pods

## What is a Pod?
A Pod is the smallest deployable unit in Kubernetes, representing a single instance of a running process in your cluster. It's an abstraction of a server that provides everything needed to run your application.

### Key Characteristics of Pods
- **Basic Unit**: The smallest and simplest Kubernetes object
- **IP Address**: A Pod gets its own IP address
- **Namespace Sharing**: Containers in a Pod share the same network namespace, IPC, and can share volumes
- **Lifecycle**: Pods are ephemeral and can be created, destroyed, and recreated
- **Managed by Controllers**: Typically managed by higher-level controllers like Deployments

## Naked Pods vs Managed Pods

### Naked Pods
- Created directly without a controller
- Not recommended for production
- Limitations:
  - No rescheduling if the node fails
  - No rolling updates
  - No scaling capabilities
  - No self-healing

### Managed Pods (Recommended)
- Created and managed by controllers (Deployments, StatefulSets, etc.)
- Benefits:
  - Automatic rescheduling
  - Rolling updates
  - Scaling capabilities
  - Self-healing

## Working with Pods

### Creating a Pod
```bash
kubectl run my-nginx --image=nginx
```

### Viewing Pods
```bash
# List all pods in the current namespace
kubectl get pods

# Get detailed information about a specific pod
kubectl describe pod my-nginx

# View pod details in YAML format
kubectl get pod my-nginx -o yaml
```

### Pod States
- **Pending**: Pod has been accepted but not all containers are running
- **Running**: All containers have been created and at least one is running
- **Succeeded**: All containers have terminated successfully
- **Failed**: At least one container has terminated in failure
- **Unknown**: The pod state could not be obtained

## Pod Lifecycle
1. **Pending**: Pod is being scheduled
2. **ContainerCreating**: Containers are being created
3. **Running**: At least one container is running
4. **Terminating**: Pod is being deleted

## Best Practices
- Always use controllers (like Deployments) to manage Pods in production
- Keep Pods as lightweight as possible (single responsibility principle)
- Use proper resource requests and limits
- Implement proper health checks (liveness and readiness probes)
- Use proper labels and selectors for organization

## Common Issues and Troubleshooting

### Checking Pod Logs
```bash
kubectl logs <pod-name>

# For multi-container pods
kubectl logs <pod-name> -c <container-name>
```

### Debugging Pods
```bash
# Get detailed pod information
kubectl describe pod <pod-name>

# Execute a command in a running pod
kubectl exec -it <pod-name> -- /bin/sh

# View pod events
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Common Problems
1. **ImagePullBackOff**: Cannot pull container image
   - Check image name and tag
   - Verify image registry access
   - Check network connectivity

2. **CrashLoopBackOff**: Container keeps crashing
   - Check container logs
   - Verify application configuration
   - Check resource constraints

3. **Pending**: Pod not scheduled
   - Check resource availability
   - Check node selectors and taints/tolerations
   - Check PersistentVolumeClaims (if any)

## Next Steps
- Learn about Pod configuration using YAML
- Understand how to configure multi-container Pods
- Explore Pod networking and storage options

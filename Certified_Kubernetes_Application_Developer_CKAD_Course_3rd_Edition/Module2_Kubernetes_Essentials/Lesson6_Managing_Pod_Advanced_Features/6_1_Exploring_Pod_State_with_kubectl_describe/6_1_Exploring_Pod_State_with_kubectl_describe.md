# 6.1 Exploring Pod State with kubectl describe

## Introduction to `kubectl describe`

`kubectl describe` is a powerful command that allows you to explore any resource in Kubernetes. When working with pods, it reveals all parameters and their current settings as stored in etcd.

### Key Features
- Presents information in a human-readable format (as opposed to raw JSON in etcd)
- Shows both user-configured settings and default values
- Provides a comprehensive view of the pod's configuration and status

## Basic Usage

```bash
# Basic syntax
kubectl describe <resource-type> <resource-name>

# Example for a pod
kubectl describe pod my-pod

# You can use either 'pod' or 'pods' (Kubernetes is case-insensitive here)
kubectl describe pods my-pod
```

## Understanding the Output

The `kubectl describe` output includes several sections:

1. **Basic Information**
   - Name
   - Namespace
   - Node where the pod is running
   - Start time
   - Labels and annotations

2. **Status**
   - Current phase (Pending, Running, Succeeded, Failed, Unknown)
   - IP address
   - Container statuses

3. **Containers**
   - Container name
   - Container ID
   - Image
   - Ports
   - Environment variables
   - Mounts
   - Resource requests and limits

4. **Conditions**
   - PodScheduled
   - Initialized
   - ContainersReady
   - Ready

5. **Events**
   - Chronological list of events related to the pod
   - Useful for troubleshooting

## Practical Examples

### Exploring a Pod's Configuration
```bash
kubectl describe pod nginx
```

### Finding Specific Information
```bash
# Find environment variables
kubectl describe pod my-pod | grep -A 10 "Environment:"

# Check resource requests and limits
kubectl describe pod my-pod | grep -A 5 "Resources"

# View container status
kubectl describe pod my-pod | grep -A 5 "Containers"
```

### Using with Other Resources
`kubectl describe` works with any Kubernetes resource:
```bash
kubectl describe service my-service
kubectl describe deployment my-deployment
kubectl describe node my-node
```

## Troubleshooting with `kubectl describe`

### Common Issues to Look For
1. **Pending Pods**
   - Check events for scheduling failures
   - Look for resource constraints or node selectors

2. **CrashLoopBackOff**
   - Check container exit codes
   - Review container logs

3. **Image Pull Errors**
   - Authentication issues
   - Image not found
   - Network connectivity problems

### Example: Debugging a Failing Pod
```bash
# Get basic pod status
kubectl get pods

# Describe the pod for detailed information
kubectl describe pod failing-pod

# Look for events at the bottom of the output
kubectl describe pod failing-pod | tail -n 20

# Check container logs
kubectl logs failing-pod

# For multi-container pods, specify the container
kubectl logs failing-pod -c container-name
```

## Best Practices

1. **Always Start with `kubectl get`**
   ```bash
   # Get an overview first
   kubectl get pods
   ```

2. **Use `-o wide` for More Context**
   ```bash
   kubectl get pods -o wide
   ```

3. **Combine with `grep` for Filtering**
   ```bash
   # Find pods in Error state
   kubectl get pods --all-namespaces | grep Error
   ```

4. **Use `-A` for All Namespaces**
   ```bash
   kubectl describe pod my-pod -A
   ```

5. **Check Events Chronologically**
   ```bash
   kubectl get events --sort-by='.metadata.creationTimestamp'
   ```

## Advanced Usage

### Field Selectors
```bash
# Describe pods with a specific label
kubectl describe pods -l app=nginx

# Describe pods in a specific phase
kubectl get pods --field-selector=status.phase=Running
```

### Output Formatting
```bash
# Output in YAML format
kubectl get pod my-pod -o yaml

# Output in JSON format
kubectl get pod my-pod -o json

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

### Watching Changes
```bash
# Watch pod changes in real-time
kubectl get pods -w

# Watch events in real-time
kubectl get events -w
```

## Common Pitfalls

1. **Too Much Information**
   - The output can be overwhelming
   - Use `grep` or `jq` to filter relevant information

2. **Namespace Issues**
   - Always specify the namespace if not using the default
   ```bash
   kubectl describe pod my-pod -n my-namespace
   ```

3. **Permission Issues**
   - Ensure you have the necessary permissions
   ```bash
   # Check your permissions
   kubectl auth can-i describe pods
   ```

4. **Resource Not Found**
   - Double-check the resource name and type
   ```bash
   # List all resources to verify
   kubectl get all
   ```

## Next Steps
- Learn about `kubectl logs` for container output
- Explore `kubectl exec` for container inspection
- Understand pod lifecycle and states
- Study Kubernetes events and monitoring

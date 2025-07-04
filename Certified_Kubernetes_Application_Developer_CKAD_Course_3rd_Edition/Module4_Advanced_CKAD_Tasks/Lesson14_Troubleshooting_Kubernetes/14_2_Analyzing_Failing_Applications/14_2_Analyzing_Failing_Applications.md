# 14.2 Analyzing Failing Applications

## Understanding Pod States

When troubleshooting applications in Kubernetes, the first step is to understand the current state of your pods. Here are the common pod states and what they mean:

### Common Pod States

| State | Description | Common Causes |
|-------|-------------|---------------|
| **Pending** | The pod has been accepted by the system, but one or more containers have not been created | - Insufficient resources<br>- Image pull issues<br>- Scheduling conflicts |
| **ContainerCreating** | The pod is being created but not all containers are running yet | - Pulling container images<br>- Creating container environment |
| **Running** | The pod is running with all containers operational | - Normal state |
| **Completed** | All containers have terminated successfully | - Job completed successfully<br>- Pod ran to completion |
| **Error** | The pod failed to start or encountered an error | - Application error<br>- Missing dependencies |
| **CrashLoopBackOff** | The container is repeatedly crashing and Kubernetes is backing off restarts | - Application crashes on startup<br>- Misconfiguration |
| **ImagePullBackOff** | Kubernetes is unable to pull the container image | - Invalid image name<br>- Authentication issues<br>- Network problems |
| **ErrImagePull** | Failed to pull the container image | - Image doesn't exist<br>- Permission denied<br>- Invalid image reference |
| **Terminating** | The pod is in the process of being terminated | - Normal during deletion<br>- May hang if there are issues |
| **Unknown** | The pod state cannot be determined | - Node communication issues<br>- Kubelet problems |

## Step-by-Step Troubleshooting Process

### 1. Check Pod Status

```bash
# Get basic pod information
kubectl get pods

# Get more detailed information
kubectl get pods -o wide

# Watch pod status in real-time
kubectl get pods -w
```

### 2. Examine Pod Details

```bash
# Get detailed information about a pod
kubectl describe pod <pod-name>

# Focus on specific sections
kubectl describe pod <pod-name> | grep -A 10 "Conditions:"
kubectl describe pod <pod-name> | grep -A 10 "Events:"
```

### 3. Check Container Logs

```bash
# View logs from all containers in the pod
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>

# View logs from a specific container
kubectl logs <pod-name> -c <container-name>

# View logs from the previous container instance (if crashed)
kubectl logs <pod-name> --previous

# Get logs with timestamps
kubectl logs <pod-name> --timestamps=true
```

### 4. Common Scenarios and Solutions

#### Scenario 1: Container Exits Immediately

**Symptoms:**
- Pod shows `Completed` state
- Container runs and exits immediately

**Troubleshooting:**
```bash
# Check container exit code
kubectl describe pod <pod-name> | grep -A 10 "State"

# View logs from the last run
kubectl logs <pod-name> --previous

# Common fix: Add a command to keep the container running
# Example for nginx:
# command: ["/bin/sh", "-c", "nginx -g 'daemon off;'"]
```

#### Scenario 2: CrashLoopBackOff

**Symptoms:**
- Pod shows `CrashLoopBackOff` state
- Container starts, crashes, and restarts in a loop

**Troubleshooting:**
```bash
# Check container logs
kubectl logs <pod-name> --previous

# Check container exit code
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'

# Common causes:
# 1. Application error - check logs
# 2. Missing configuration - check environment variables and volumes
# 3. Resource constraints - check resource limits
# 4. Liveness/readiness probe failures
```

#### Scenario 3: Image Pull Errors

**Symptoms:**
- Pod shows `ImagePullBackOff` or `ErrImagePull` state
- Container fails to start due to image issues

**Troubleshooting:**
```bash
# Check the exact error message
kubectl describe pod <pod-name> | grep -i error -A 5

# Common fixes:
# 1. Verify image name and tag
# 2. Check if image exists in the registry
# 3. Configure image pull secrets if needed:
#    kubectl create secret docker-registry <secret-name> \
#      --docker-server=<registry-server> \
#      --docker-username=<username> \
#      --docker-password=<password>
```

### 5. Debugging Techniques

#### Exec into a Running Container

```bash
# Get an interactive shell in the container
kubectl exec -it <pod-name> -- /bin/sh

# Or for older images
kubectl exec -it <pod-name> -- /bin/bash

# Run a specific command in the container
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- ps aux
```

#### Copy Files to/from Containers

```bash
# Copy file from pod to local
kubectl cp <pod-name>:/path/to/remote/file /path/to/local/file

# Copy file from local to pod
kubectl cp /path/to/local/file <pod-name>:/path/to/remote/file
```

### 6. Debugging Init Containers

```bash
# View init container status
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].state}'

# View init container logs
kubectl logs <pod-name> -c <init-container-name>

# Common init container issues:
# 1. Timeout waiting for dependencies
# 2. Permission issues
# 3. Network connectivity problems
```

### 7. Resource Constraints

```bash
# Check pod resource requests/limits
kubectl get pod <pod-name> -o json | jq '.spec.containers[].resources'

# Check node resource usage
kubectl top nodes
kubectl top pods

# Common issues:
# 1. Container exceeds memory limit (OOMKilled)
# 2. Insufficient CPU resources
# 3. Resource quotas exceeded
```

## Practical Examples

### Example 1: Debugging a Failing Nginx Pod

```bash
# Create a failing nginx deployment
kubectl create deployment nginx --image=nginx:invalid-tag

# Check pod status
kubectl get pods
# Output: Shows ImagePullBackOff

# Get more details
kubectl describe pod <pod-name>
# Output: Shows "Failed to pull image" error

# Fix by updating to a valid image
kubectl set image deployment/nginx nginx=nginx:latest
```

### Example 2: Debugging a Crashing Application

```bash
# Create a pod that will crash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: crash-container
    image: busybox
    command: ['/bin/sh', '-c', 'echo "Crashing..."; exit 1']
  restartPolicy: OnFailure
EOF

# Check pod status
kubectl get pods
# Output: Shows CrashLoopBackOff

# Check logs
kubectl logs crash-pod
# Output: Shows "Crashing..."

# Check previous container logs
kubectl logs crash-pod --previous

# Fix by updating the command to not exit
kubectl edit pod crash-pod
# Change command to: ['/bin/sh', '-c', 'echo "Running..."; sleep 3600']
```

## Best Practices for Debugging

1. **Enable Debug Logging**
   - Add verbose logging to your application
   - Use structured logging for better analysis

2. **Use Sidecar Containers**
   - Add a sidecar container for debugging
   - Example: A sidecar that runs `kubectl proxy` or provides a debugging interface

3. **Implement Health Checks**
   - Add liveness and readiness probes
   - Use startup probes for slow-starting applications

4. **Monitor Resource Usage**
   - Set up resource requests and limits
   - Monitor for OOMKilled events

5. **Use Ephemeral Containers for Debugging**
   ```bash
   # Add an ephemeral debug container
   kubectl debug -it <pod-name> --image=busybox --target=<container-name>
   ```

6. **Check Node Health**
   ```bash
   # Check node status
   kubectl get nodes
   
   # Get node details
   kubectl describe node <node-name>
   
   # Check node conditions
   kubectl get nodes -o json | jq '.items[].status.conditions'
   ```

## Common Exit Codes and Their Meanings

| Exit Code | Meaning | Common Causes |
|-----------|---------|---------------|
| 0 | Success | Normal termination |
| 1 | General error | Application error, missing file |
| 126 | Permission problem | File not executable |
| 127 | Command not found | Missing executable in PATH |
| 128 + n | Fatal error signal | Killed by signal n |
| 130 | Interrupted | SIGINT (Ctrl+C) |
| 137 | Killed | SIGKILL (OOMKilled) |
| 143 | Terminated | SIGTERM (graceful shutdown) |

## Tools for Advanced Debugging

1. **kubectl debug**
   ```bash
   # Create a copy of the pod with debugging tools
   kubectl debug -it <pod-name> --image=busybox --copy-to=debug-pod
   ```

2. **kubectl port-forward**
   ```bash
   # Forward a local port to a pod
   kubectl port-forward <pod-name> 8080:80
   ```

3. **kubectl proxy**
   ```bash
   # Start a proxy to the Kubernetes API
   kubectl proxy --port=8001
   # Then access API at http://localhost:8001/api/
   ```

4. **kubectl top**
   ```bash
   # Show resource usage
   kubectl top pods
   kubectl top nodes
   ```

By following these steps and using these tools, you can effectively diagnose and resolve issues with failing applications in Kubernetes.

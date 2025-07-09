# CKAD Practice Question 16: Sidecar Containers and Log Management

## Question
The tech lead of Mercury2D has decided to implement additional logging to investigate missing data incidents. There's an existing deployment with the following details:

**Existing Resources**:
- Deployment: `cleaner`
- Namespace: `mercury`
- Existing Container: `cleaner-con`
  - Mounts a volume
  - Writes logs to `/var/log/cleaner.log`

**Tasks**:
1. **Add a Sidecar Container**:
   - Name: `logger-con`
   - Image: `busybox:1.31.0`
   - Mount the same volume as `cleaner-con`
   - Continuously stream the contents of `cleaner.log` to stdout using `tail -f`

2. **Investigate Logs**:
   - Check the logs of the new container for information about missing data incidents
   - The deployment YAML is available at `/opt/course/16/cleaner.yaml`
   - Save your changes to `/opt/course/16/cleaner-new.yaml`

## Solution

### Step 1: Examine the Existing Deployment
```bash
# Check the existing deployment and pods
kubectl -n mercury get deployment cleaner
kubectl -n mercury get pods -l app=cleaner

# View the current deployment configuration
kubectl -n mercury get deployment cleaner -o yaml > /opt/course/16/cleaner.yaml
```

### Step 2: Create an Updated Deployment with Sidecar
Edit `/opt/course/16/cleaner-new.yaml` to add the sidecar container:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cleaner
  namespace: mercury
  labels:
    app: cleaner
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cleaner
  template:
    metadata:
      labels:
        app: cleaner
    spec:
      containers:
      - name: cleaner-con
        image: nginx:1.19.0
        volumeMounts:
        - name: log-volume
          mountPath: /var/log
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo 'Remove random file' >> /var/log/cleaner.log; sleep 5; done"]
      
      # Add the sidecar container
      - name: logger-con
        image: busybox:1.31.0
        command: ["sh", "-c"]
        args: ["tail -f /var/log/cleaner.log"]
        volumeMounts:
        - name: log-volume
          mountPath: /var/log
      
      volumes:
      - name: log-volume
        emptyDir: {}
```

### Step 3: Apply the Updated Deployment
```bash
# Apply the updated deployment
kubectl apply -f /opt/course/16/cleaner-new.yaml -n mercury

# Verify the pods are running with two containers each
kubectl -n mercury get pods -l app=cleaner
```

### Step 4: Check the Logs
```bash
# Get the name of a pod
POD_NAME=$(kubectl -n mercury get pods -l app=cleaner -o name | head -n 1)

# View logs from the sidecar container
kubectl -n mercury logs $POD_NAME -c logger-con
```

### Step 5: Verify the Solution
```bash
# Check the deployment status
kubectl -n mercury describe deployment cleaner

# Verify both containers are running in each pod
kubectl -n mercury get pods -l app=cleaner -o jsonpath='{range .items[*]}{.metadata.name}: {.status.containerStatuses[*].name}{"\n"}{end}'
```

## Explanation

### Key Concepts
1. **Sidecar Containers**: Additional containers in a pod that enhance or extend the functionality of the main container
2. **Shared Volumes**: Allows multiple containers in the same pod to share files
3. **Log Management**: Using sidecar containers to handle logs from the main application
4. **EmptyDir Volumes**: Temporary storage that exists for the lifetime of the pod

### Why This Solution Works
- The sidecar container mounts the same volume as the main container
- It uses `tail -f` to continuously stream the log file to stdout
- The logs can be accessed using `kubectl logs` on the sidecar container
- The solution follows the pattern of separating logging concerns from the main application

### Exam Tips
1. **Volume Sharing**: Ensure both containers mount the volume at the same path
2. **Container Naming**: Use meaningful names for sidecar containers
3. **Resource Limits**: Consider adding resource limits to sidecar containers
4. **Log Rotation**: In production, implement log rotation to prevent excessive disk usage
5. **Debugging**: Use `kubectl describe pod` to verify volume mounts

### Common Mistakes to Avoid
- Forgetting to mount the volume in the sidecar container
- Using different mount paths in the containers
- Not setting the correct command/args for the sidecar
- Overlooking resource constraints when adding sidecar containers

## Additional Practice
1. Add resource limits to the sidecar container
2. Configure log rotation for the log file
3. Create a sidecar that forwards logs to a centralized logging system
4. Add liveness and readiness probes to the sidecar
5. Implement a sidecar that processes logs in real-time

## Related Commands
```bash
# View logs from a specific container in a pod
kubectl logs <pod-name> -c <container-name> -n <namespace>

# Stream logs in real-time
kubectl logs -f <pod-name> -c <container-name> -n <namespace>

# Get detailed information about a pod
kubectl describe pod <pod-name> -n <namespace>

# Execute a command in a container
kubectl exec -it <pod-name> -c <container-name> -n <namespace> -- <command>

# View container logs with timestamps
kubectl logs --previous <pod-name> -c <container-name> -n <namespace>
```

# CKAD Practice Question 17: Init Containers

## Question
You need to demonstrate the use of an init container to a coworker. There's an existing deployment with the following details:

**Existing Resources**:
- Deployment YAML location: `/opt/course/17/deployment.yaml`
- Namespace: `mars`
- Main container: `nginx` (image: `nginx:alpine`)
  - Serves files from a mounted volume
  - The volume is currently empty
  - Exposes port 80

**Tasks**:
1. **Add an Init Container**:
   - Name: `init-con`
   - Image: `busybox:1.31.0`
   - Mount the same volume as the main container
   - Create a file `index.html` with the content "Check this out" in the root of the mounted volume

2. **Test the Implementation**:
   - Use a temporary nginx:alpine pod to test access to the web server
   - Verify the content is being served correctly

## Solution

### Step 1: Examine the Existing Deployment
```bash
# Check the existing deployment
kubectl -n mars get deployment

# View the current deployment configuration
kubectl -n mars get deployment webapp -o yaml > /opt/course/17/deployment.yaml
```

### Step 2: Create an Updated Deployment with Init Container
Edit `/opt/course/17/deployment-new.yaml` to add the init container:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: mars
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      # Add init container
      initContainers:
      - name: init-con
        image: busybox:1.31.0
        command: ['sh', '-c']
        args: ['echo "Check this out" > /usr/share/nginx/html/index.html']
        volumeMounts:
        - name: webcontent
          mountPath: /usr/share/nginx/html
      
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: webcontent
          mountPath: /usr/share/nginx/html
      
      volumes:
      - name: webcontent
        emptyDir: {}
```

### Step 3: Apply the Updated Deployment
```bash
# Apply the updated deployment
kubectl apply -f /opt/course/17/deployment-new.yaml -n mars

# Verify the pod is running with the init container
kubectl -n mars get pods -l app=webapp
```

### Step 4: Test the Web Server
```bash
# Get the pod IP address
POD_IP=$(kubectl -n mars get pods -l app=webapp -o jsonpath='{.items[0].status.podIP}')

# Create a temporary pod to test the web server
kubectl run test-curl --image=nginx:alpine --rm -it --restart=Never -- \
  sh -c "apk add --no-cache curl && curl http://$POD_IP"
```

### Step 5: Verify the Solution
```bash
# Check the pod's init container status
kubectl -n mars describe pod -l app=webapp | grep -A 5 "Init Containers"

# Verify the file was created in the volume
kubectl -n mars exec -it $(kubectl -n mars get pods -l app=webapp -o name | head -n 1) -- \
  ls -la /usr/share/nginx/html/
```

## Explanation

### Key Concepts
1. **Init Containers**: Specialized containers that run before app containers in a pod
2. **Volume Sharing**: Multiple containers in a pod can share the same volume
3. **Container Lifecycle**: Init containers must complete successfully before app containers start
4. **EmptyDir Volumes**: Temporary storage that exists for the lifetime of the pod

### Why This Solution Works
- The init container runs before the main nginx container
- It mounts the same volume at the same path as the main container
- The command creates an `index.html` file with the required content
- The main nginx container starts only after the init container completes successfully
- The volume persists the file for the main container to use

### Exam Tips
1. **Order Matters**: Init containers run in the order they're defined
2. **Completion**: All init containers must complete successfully for the pod to start
3. **Debugging**: Use `kubectl describe pod` to check init container status
4. **Resource Sharing**: Volumes are the recommended way to share data between containers in a pod
5. **Cleanup**: Init containers are automatically removed after completion

### Common Mistakes to Avoid
- Forgetting to mount the volume in both containers
- Using different mount paths in the containers
- Not setting the correct command/args for the init container
- Not checking init container logs when debugging pod startup issues

## Additional Practice
1. Add multiple init containers with dependencies between them
2. Configure resource limits for init containers
3. Use init containers to clone git repositories
4. Implement startup probes for init containers
5. Create a pod with both init containers and sidecar containers

## Related Commands
```bash
# View init container logs
kubectl logs <pod-name> -c <init-container-name> -n <namespace>

# Get detailed pod information including init containers
kubectl describe pod <pod-name> -n <namespace>

# View pod events
kubectl get events --field-selector involvedObject.name=<pod-name> -n <namespace>

# Check container statuses
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[*].state}' -n <namespace>

# Check init container statuses
kubectl get pod <pod-name> -o jsonpath='{.status.initContainerStatuses[*].state}' -n <namespace>
```

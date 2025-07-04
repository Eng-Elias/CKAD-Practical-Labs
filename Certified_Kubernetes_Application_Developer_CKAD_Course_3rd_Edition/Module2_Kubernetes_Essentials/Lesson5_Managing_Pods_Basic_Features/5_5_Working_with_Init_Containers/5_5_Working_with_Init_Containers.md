# 5.5 Working with Init Containers

## What are Init Containers?

Init containers are specialized containers that run before the main application containers in a pod. They are designed to perform setup tasks that must complete successfully before the main containers start.

### Key Characteristics
- Run to completion before app containers start
- Run in sequence (one after another)
- Must complete successfully before the next init container starts
- If any init container fails, the pod restarts (following the pod's restart policy)
- Have their own resource limits and requests

## When to Use Init Containers

### Common Use Cases
1. **Dependency Checks**: Wait for a service to be available
2. **Configuration Download**: Fetch configuration from a remote source
3. **Database Migrations**: Run database schema migrations
4. **Secret Injection**: Retrieve and inject secrets
5. **Data Pre-loading**: Download required data or assets

## Basic Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  # Regular containers
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  
  # Init containers
  initContainers:
  - name: install
    image: busybox:1.28
    command: ['sh', '-c', 'echo "<h1>Hello from the init container</h1>" > /work-dir/index.html']
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  
  # Shared volume
  volumes:
  - name: workdir
    emptyDir: {}
```

## Multiple Init Containers

When multiple init containers are specified, they run in sequence:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo-multiple
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  
  initContainers:
  # First init container
  - name: init-1
    image: busybox:1.28
    command: ['sh', '-c', 'echo "<h1>Init Container 1</h1>" > /work-dir/index.html']
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  
  # Second init container (runs after first completes)
  - name: init-2
    image: busybox:1.28
    command: ['sh', '-c', 'echo "<h1>Init Container 2</h1>" >> /work-dir/index.html']
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  
  volumes:
  - name: workdir
    emptyDir: {}
```

## Real-World Examples

### 1. Wait for a Service to Be Available

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  
  initContainers:
  - name: wait-for-db
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mysql-service; do echo waiting for mysql service; sleep 2; done']
```

### 2. Clone a Git Repository

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: git-clone-example
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    volumeMounts:
    - name: git-repo
      mountPath: /usr/share/nginx/html
  
  initContainers:
  - name: git-clone
    image: alpine/git
    command: ['git', 'clone', 'https://github.com/example/website.git', '/repo']
    volumeMounts:
    - name: git-repo
      mountPath: /repo
  
  volumes:
  - name: git-repo
    emptyDir: {}
```

## Advanced Init Container Features

### 1. Resource Limits
```yaml
initContainers:
- name: init-resource-demo
  image: busybox:1.28
  command: ['sh', '-c', 'echo "Setting up..." && sleep 30']
  resources:
    requests:
      cpu: "100m"
      memory: "64Mi"
    limits:
      cpu: "200m"
      memory: "128Mi"
```

### 2. Security Context
```yaml
initContainers:
- name: init-security-context
  image: busybox:1.28
  command: ['sh', '-c', 'echo "Running with non-root user"']
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
```

## Debugging Init Containers

### Checking Init Container Status
```bash
# Get pod status including init containers
kubectl get pod <pod-name> -o wide

# Describe pod to see init container status
kubectl describe pod <pod-name>

# View logs of an init container
kubectl logs <pod-name> -c <init-container-name>

# View previous logs if init container crashed
kubectl logs <pod-name> -c <init-container-name> --previous
```

### Common Issues
1. **CrashLoopBackOff**: The init container is failing and Kubernetes keeps restarting it
2. **ImagePullBackOff**: Cannot pull the init container image
3. **Init:Error**: The init container failed with a non-zero exit code
4. **PodInitializing**: The pod is still initializing (normal state during initialization)

## Best Practices

1. **Keep Init Containers Lightweight**: They should only perform necessary setup
2. **Set Timeouts**: Use `activeDeadlineSeconds` to prevent hanging
3. **Handle Failures**: Implement proper error handling in your init scripts
4. **Use Small Images**: Prefer minimal base images to reduce startup time
5. **Log Everything**: Ensure init containers log their progress and errors
6. **Idempotency**: Make init containers idempotent when possible

## Complete Example with All Features

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complete-init-example
  labels:
    app: myapp
    environment: production
spec:
  # Pod-level security context
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  
  # Active deadline for all init containers (in seconds)
  activeDeadlineSeconds: 300
  
  # Init containers run in sequence
  initContainers:
  # Wait for database to be ready
  - name: wait-for-db
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mysql-service; do echo "Waiting for database..."; sleep 2; done']
    resources:
      requests:
        cpu: "50m"
        memory: "32Mi"
    securityContext:
      runAsUser: 1000
      runAsNonRoot: true
  
  # Clone configuration repository
  - name: clone-config
    image: alpine/git
    command: ['git', 'clone', 'https://github.com/example/config-repo.git', '/config']
    volumeMounts:
    - name: config
      mountPath: /config
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
  
  # Process configuration
  - name: process-config
    image: busybox:1.28
    command: ['sh', '-c', 'cp /config/app-config.yaml /processed-config/ && echo "Processing complete"']
    volumeMounts:
    - name: config
      mountPath: /config
      readOnly: true
    - name: processed-config
      mountPath: /processed-config
  
  # Main application container
  containers:
  - name: app
    image: myapp:1.0.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: processed-config
      mountPath: /app/config
    - name: data
      mountPath: /data
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
  
  # Volumes
  volumes:
  - name: config
    emptyDir: {}
  - name: processed-config
    emptyDir: {}
  - name: data
    emptyDir:
      sizeLimit: 1Gi
```

## Next Steps
- Learn about Pod Lifecycle Hooks
- Explore Pod Presets for common configurations
- Understand Pod Topology Spread Constraints
- Study Pod Disruption Budgets

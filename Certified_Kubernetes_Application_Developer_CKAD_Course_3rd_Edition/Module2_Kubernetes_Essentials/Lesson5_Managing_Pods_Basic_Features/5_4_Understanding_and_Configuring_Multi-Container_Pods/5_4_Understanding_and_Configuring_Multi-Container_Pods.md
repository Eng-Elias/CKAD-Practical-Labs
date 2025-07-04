# 5.4 Understanding and Configuring Multi-Container Pods

## What are Multi-Container Pods?
A multi-container pod is a pod that runs multiple containers within a single pod. These containers share the same lifecycle, network namespace, and can communicate with each other using localhost.

### Key Characteristics
- **Shared Network**: Containers share the same IP address and port space
- **Shared Storage**: Can share volumes for data exchange
- **Atomic Lifecycle**: All containers in a pod are created and terminated together
- **Co-located**: Containers are always scheduled on the same node

## When to Use Multi-Container Pods

### Common Use Cases
1. **Sidecar Containers**: Add functionality to the main container (e.g., logging, monitoring)
2. **Adapter Containers**: Normalize output from the main container
3. **Ambassador Containers**: Proxy network traffic to the main container
4. **Proxies, Bridges, and Adapters**: Extend application functionality

## Multi-Container Pod Patterns

### 1. Sidecar Pattern
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-collector
    image: busybox
    args: [/bin/sh, -c, 'tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

### 2. Adapter Pattern
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-example
spec:
  containers:
  - name: app
    image: myapp
    ports:
    - containerPort: 8080
  - name: adapter
    image: adapter-image
    command: ["/bin/sh", "-c", "while true; do curl -s http://localhost:8080/metrics | transform-metrics > /var/metrics/metrics.txt; sleep 30; done"]
    volumeMounts:
    - name: metrics
      mountPath: /var/metrics
  volumes:
  - name: metrics
    emptyDir: {}
```

### 3. Ambassador Pattern
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-example
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DB_HOST
      value: localhost
    - name: DB_PORT
      value: "3306"
  - name: db-proxy
    image: envoyproxy/envoy
    # Proxy configuration would go here
```

## Configuring Multi-Container Pods

### 1. Shared Volumes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-example
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "Hello from writer" > /shared/log.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /shared/log.txt && sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  volumes:
  - name: shared-data
    emptyDir: {}
```

### 2. Inter-Container Communication
Containers in the same pod can communicate using `localhost`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: communication-example
spec:
  containers:
  - name: web
    image: nginx
  - name: logger
    image: busybox
    command: ['sh', '-c', 'while true; do curl -s http://localhost > /dev/null; echo "Accessed web server at $(date)"; sleep 10; done']
```

## Managing Multi-Container Pods

### Viewing Containers in a Pod
```bash
# List all pods
kubectl get pods

# Get detailed information about a specific pod
kubectl describe pod <pod-name>

# Get logs from a specific container
kubectl logs <pod-name> -c <container-name>

# Execute a command in a specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

### Resource Management
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-example
spec:
  containers:
  - name: app
    image: myapp
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: sidecar
    image: sidecar-image
    resources:
      requests:
        memory: "32Mi"
        cpu: "100m"
      limits:
        memory: "64Mi"
        cpu: "200m"
```

## Common Issues and Solutions

### 1. Container Startup Order
Kubernetes doesn't guarantee container startup order. Use init containers if you need to ensure certain containers start before others.

### 2. Port Conflicts
Containers in the same pod cannot use the same port. Each container must use unique ports.

### 3. Resource Starvation
Monitor resource usage to ensure one container doesn't starve others.

### 4. Debugging
```bash
# Check pod events
kubectl describe pod <pod-name>

# View logs from all containers
kubectl logs <pod-name> --all-containers

# Check container status
kubectl get pods -o jsonpath='{range .status.containerStatuses[*]}{.name}: {.state}{"\n"}{end}' <pod-name>
```

## Best Practices

1. **Single Responsibility**: Each container should have a single responsibility
2. **Minimal Containers**: Keep containers minimal and focused
3. **Resource Limits**: Always set resource requests and limits
4. **Health Checks**: Implement liveness and readiness probes
5. **Logging**: Ensure each container logs to stdout/stderr
6. **Documentation**: Document the purpose of each container

## Example: Complete Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complete-example
  labels:
    app: myapp
    tier: frontend
spec:
  # Init containers run before the main containers
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nslookup db-service; do echo waiting for db; sleep 2; done']
  
  # Main containers
  containers:
  # Main application container
  - name: web
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d
    - name: shared-data
      mountPath: /usr/share/nginx/html
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
  
  # Sidecar container for logging
  - name: log-collector
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log/nginx
    resources:
      requests:
        cpu: "50m"
        memory: "32Mi"
      limits:
        cpu: "100m"
        memory: "64Mi"
  
  # Volumes shared between containers
  volumes:
  - name: config
    configMap:
      name: nginx-config
  - name: logs
    emptyDir: {}
  - name: shared-data
    emptyDir: {}
```

## Next Steps
- Learn about Init Containers for pod initialization
- Explore Pod Presets for injecting common configurations
- Understand Pod Security Policies
- Study Pod Lifecycle Hooks

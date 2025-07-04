# 10.2 Configuring Pod Volume Storage

## Introduction to Pod Volumes

Pod volumes provide a way to store data that persists beyond the lifecycle of a container. They are defined in the pod specification and can be mounted into containers within the pod.

## Volume Types for Pod Storage

### 1. emptyDir
- **Purpose**: Temporary storage that exists for the lifetime of the pod
- **Use Cases**:
  - Scratch space
  - Checkpointing long computations
  - Sharing files between containers in a pod
- **Example**:
  ```yaml
  volumes:
    - name: cache-volume
      emptyDir: {}
  ```

### 2. hostPath
- **Purpose**: Mounts a file or directory from the host node
- **Use Cases**:
  - Accessing node logs
  - Accessing the Docker daemon
  - Node-specific data
- **Example**:
  ```yaml
  volumes:
    - name: host-path-volume
      hostPath:
        path: /data
        type: Directory
  ```

## Creating a Pod with Volumes

### Basic Pod with emptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do date >> /cache/date.log; sleep 5; done"]
    volumeMounts:
    - name: cache-volume
      mountPath: /cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### Multi-Container Pod with Shared Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "while true; do date >> /shared/date.log; sleep 5; done"]
    volumeMounts:
    - name: shared-volume
      mountPath: /shared
  
  - name: reader
    image: busybox
    command: ["sh", "-c", "tail -f /shared/date.log"]
    volumeMounts:
    - name: shared-volume
      mountPath: /shared
  
  volumes:
  - name: shared-volume
    emptyDir: {}
```

## Volume Mount Options

### Mount Propagation
- **None**: The volume mount is private
- **HostToContainer**: Mounts mounted inside the volume are visible to the container
- **Bidirectional**: Container mounts are propagated back to the host

### Read-Only Volumes
```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config
  readOnly: true
```

### Subpath Mounting
Mount a specific file or subdirectory from a volume:
```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/nginx/nginx.conf
  subPath: nginx.conf
```

## Practical Example: Web Server with Config

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d/default.conf
      subPath: default.conf
    - name: html
      mountPath: /usr/share/nginx/html
  
  volumes:
  - name: config
    configMap:
      name: nginx-config
  
  - name: html
    emptyDir: {}
```

## Best Practices

1. **Use emptyDir for Temporary Storage**
   - Ideal for caching and temporary files
   - Data is lost when pod is removed

2. **Use hostPath with Caution**
   - Ties pods to specific nodes
   - Can cause issues with pod scheduling
   - Consider using PersistentVolumes for production

3. **Set Resource Limits**
   - Use size limits for emptyDir
   - Example:
     ```yaml
     volumes:
     - name: cache
       emptyDir:
         sizeLimit: 500Mi
     ```

4. **Security Context**
   - Set appropriate file permissions
   - Use securityContext for non-root users

## Troubleshooting

### Common Issues
1. **Permission Denied**
   - Check volume permissions
   - Verify securityContext settings

2. **Volume Mount Fails**
   - Check if volume exists in pod spec
   - Verify mount paths don't conflict

3. **Storage Not Persistent**
   - emptyDir is not persistent across pod restarts
   - Use PersistentVolumes for true persistence

### Diagnostic Commands
```bash
# Check pod status and events
kubectl describe pod <pod-name>

# Check container logs
kubectl logs <pod-name> -c <container-name>

# Execute commands in container
kubectl exec -it <pod-name> -- ls /mount/path

# Check volume mounts
kubectl get pod <pod-name> -o jsonpath='{.spec.volumes}'
```

## Next Steps
In the next section, we'll explore Persistent Volumes (PVs) and how they provide cluster-wide persistent storage.

# 10.5 Configuring Pod Storage with PV and PVC

## Introduction

This section demonstrates how to configure pod storage using both Persistent Volumes (PV) and Persistent Volume Claims (PVC) together. This is the recommended approach for managing persistent storage in Kubernetes.

## Complete Storage Workflow

1. **Admin creates a Persistent Volume (PV)**
   - Defines the actual storage
   - Specifies capacity and access modes

2. **User creates a Persistent Volume Claim (PVC)**
   - Requests storage with specific requirements
   - Binds to an available PV

3. **Pod uses the PVC**
   - Mounts the PVC as a volume
   - Applications read/write to the mounted path

## Example: Web Application with Persistent Storage

### 1. Create a Persistent Volume (PV)

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-data-pv
  labels:
    type: local
    app: myapp
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/app"
  persistentVolumeReclaimPolicy: Retain
```

### 2. Create a Persistent Volume Claim (PVC)

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  selector:
    matchLabels:
      app: myapp
```

### 3. Create a Pod that uses the PVC

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
  labels:
    app: web
spec:
  containers:
  - name: web
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: app-storage
      mountPath: "/usr/share/nginx/html"
  volumes:
  - name: app-storage
    persistentVolumeClaim:
      claimName: app-data-pvc
```

## Verifying the Setup

1. **Check PV and PVC Status**
   ```bash
   kubectl get pv
   kubectl get pvc
   ```

2. **Check Pod Status**
   ```bash
   kubectl get pods
   kubectl describe pod web-app
   ```

3. **Test Persistent Storage**
   ```bash
   # Create a test file
   kubectl exec -it web-app -- sh -c "echo 'Hello, Persistent Storage!' > /usr/share/nginx/html/test.txt"
   
   # Verify the file exists
   kubectl exec -it web-app -- cat /usr/share/nginx/html/test.txt
   
   # Delete the pod
   kubectl delete pod web-app
   
   # Recreate the pod
   kubectl apply -f pod.yaml
   
   # Verify the file still exists
   kubectl exec -it web-app -- cat /usr/share/nginx/html/test.txt
   ```

## Multi-Container Pod with Shared Storage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-app
spec:
  containers:
  - name: writer
    image: alpine
    command: ["sh", "-c", "while true; do echo $(date) >> /data/log.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-storage
      mountPath: /data
  
  - name: reader
    image: alpine
    command: ["sh", "-c", "tail -f /data/log.txt"]
    volumeMounts:
    - name: shared-storage
      mountPath: /data
  
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: app-data-pvc
```

## Stateful Applications with Persistent Storage

### StatefulSet with PVC Template

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Best Practices

1. **Use Meaningful Names**
   - PV: `{app}-{purpose}-pv` (e.g., `wordpress-data-pv`)
   - PVC: `{app}-{purpose}-pvc` (e.g., `wordpress-data-pvc`)

2. **Resource Management**
   - Set appropriate storage sizes
   - Monitor disk usage
   - Implement resource quotas

3. **Security**
   - Use proper file permissions
   - Consider security contexts
   - Use network policies for network storage

4. **Backup Strategy**
   - Regular backups of important data
   - Test restore procedures
   - Consider volume snapshots

## Troubleshooting

### Common Issues

1. **Pod Stuck in Pending**
   - Check PVC binding status
   - Verify PV is available
   - Check resource quotas

2. **Permission Denied**
   - Check file permissions
   - Verify security context
   - Check storage class settings

3. **Storage Not Persistent**
   - Verify reclaim policy
   - Check if PVC is properly bound
   - Verify mount paths

### Diagnostic Commands

```bash
# Check PV and PVC status
kubectl get pv,pvc

# Describe resources for detailed information
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name> -c <container-name>

# Check events
kubectl get events --sort-by='.metadata.creationTimestamp'
```

## Next Steps
In the next section, we'll explore StorageClass and how it enables dynamic provisioning of storage.

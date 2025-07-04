# 10.4 Configuring PVCs (Persistent Volume Claims)

## Introduction to Persistent Volume Claims

Persistent Volume Claims (PVCs) are requests for storage by users. They act as a bridge between pods and Persistent Volumes (PVs), allowing for dynamic provisioning and management of storage resources.

## PVC Lifecycle

1. **Provisioning**
   - Static: Binds to pre-created PV
   - Dynamic: Automatically provisions PV using StorageClass

2. **Binding**
   - Control loop watches for new PVCs
   - Binds to an available PV that matches requirements
   - Binding is exclusive (1:1 relationship)

3. **Using**
   - Pods reference the PVC
   - Storage is mounted into the pod

4. **Releasing**
   - User deletes the PVC
   - PV enters "Released" state

## Creating a PVC

### Basic PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### PVC with Selectors

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: myapp
    matchExpressions:
      - {key: environment, operator: In, values: [dev, test]}
  storageClassName: standard
```

## PVC Status

- **Pending**: Waiting for a PV to be created/bound
- **Bound**: Successfully bound to a PV
- **Lost**: The bound PV has been deleted
- **Terminating**: The PVC is being deleted

## Access Modes

| Access Mode | Description |
|-------------|-------------|
| ReadWriteOnce (RWO) | Read-write by a single node |
| ReadOnlyMany (ROX) | Read-only by many nodes |
| ReadWriteMany (RWX) | Read-write by many nodes |

## Storage Classes

### Using Default Storage Class

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: default-storage-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  # No storageClassName specified, uses default
```

### Explicit Storage Class

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-storage-pvc
spec:
  storageClassName: fast
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
```

## Using PVCs in Pods

### Pod with PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc
```

## PVC Updates and Resizing

### Expanding a PVC

1. Edit the PVC to request more storage:
   ```bash
   kubectl edit pvc my-pvc
   ```
   Change `spec.resources.requests.storage` to a larger value

2. Check the status:
   ```bash
   kubectl get pvc my-pvc -w
   ```

### Volume Expansion Prerequisites
- The StorageClass must have `allowVolumeExpansion: true`
- The underlying storage system must support resizing
- The PVC must be in a bound state

## Best Practices

1. **Use Meaningful Names**
   - Example: `mysql-data-pvc`, `app-logs-pvc`
   - Include purpose and application name

2. **Resource Requests**
   - Set realistic storage requirements
   - Monitor usage and adjust as needed

3. **Access Modes**
   - Choose the most restrictive mode that works
   - Prefer ReadWriteOnce when possible

4. **Storage Classes**
   - Use explicit storage classes
   - Document storage requirements

5. **Cleanup**
   - Delete PVCs when no longer needed
   - Understand reclaim policies

## Troubleshooting

### Common Issues

1. **PVC Stays Pending**
   - No available PV matches the PVC requirements
   - StorageClass doesn't exist
   - Insufficient storage capacity

2. **Binding Issues**
   - Access mode mismatch
   - Storage size mismatch
   - Node affinity conflicts

3. **Mount Failures**
   - Permission issues
   - Storage backend problems
   - Network connectivity issues

### Diagnostic Commands

```bash
# List all PVCs
kubectl get pvc

# Get detailed PVC information
kubectl describe pvc <pvc-name>

# Check events
kubectl get events --field-selector involvedObject.kind=PersistentVolumeClaim

# Check PV binding
kubectl get pv

# Check storage classes
kubectl get storageclass
```

## Example: Complete PVC Workflow

1. Create a StorageClass (if using dynamic provisioning):
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: fast
   provisioner: kubernetes.io/aws-ebs
   parameters:
     type: gp2
   allowVolumeExpansion: true
   ```

2. Create a PVC:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: myapp-data
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
     storageClassName: fast
   ```

3. Use the PVC in a Pod:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: myapp
   spec:
     containers:
     - name: myapp
       image: nginx
       volumeMounts:
       - name: data
         mountPath: /data
     volumes:
     - name: data
       persistentVolumeClaim:
         claimName: myapp-data
   ```

## Next Steps
In the next section, we'll explore how to configure pod storage with PV and PVC together in a complete example.

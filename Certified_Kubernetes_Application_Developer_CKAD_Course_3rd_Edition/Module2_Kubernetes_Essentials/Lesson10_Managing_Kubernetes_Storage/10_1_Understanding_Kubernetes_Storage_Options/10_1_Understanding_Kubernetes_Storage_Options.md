# 10.1 Understanding Kubernetes Storage Options

## Container Storage Fundamentals

### Container Storage Basics
- Containers use a read-write layer on the host
- Data is ephemeral by default - removed when container stops
- Container storage is tied to the container's lifecycle

### Volume Storage in Pods
- Volumes provide persistent storage for pods
- Data persists across container restarts
- Different volume types available (emptyDir, hostPath, cloud storage, etc.)
- Defined in the pod specification

## Kubernetes Storage Architecture

### Persistent Volumes (PV)
- Cluster-wide resource that represents storage
- Created by administrators or dynamically provisioned
- Has specific properties (size, access modes, reclaim policy)
- Independent of pod lifecycle

### Persistent Volume Claims (PVC)
- Request for storage by a user/application
- Binds to a matching PV
- Defines storage requirements (size, access modes)
- Used by pods to access persistent storage

### StorageClass
- Enables dynamic provisioning of storage
- Defines a "class" of storage
- Automatically creates PVs when PVCs are created
- Handled by provisioners specific to storage backends

## Storage Access Modes

| Access Mode | Description |
|-------------|-------------|
| ReadWriteOnce (RWO) | Read-write by a single node |
| ReadOnlyMany (ROX) | Read-only by many nodes |
| ReadWriteMany (RWX) | Read-write by many nodes |

## Common Volume Types

### emptyDir
- Temporary directory that shares pod's lifetime
- Created when pod is assigned to a node
- Deleted when pod is removed from node
- Good for temporary data, caching, or sharing between containers

### hostPath
- Mounts a file or directory from the host node
- Data persists across pod restarts
- Tied to a specific node
- Use cases: system-level pods, node-specific data

### Cloud Provider Volumes
- AWS EBS, Azure Disk, GCE Persistent Disk
- Integrates with cloud provider's storage solutions
- Handles provisioning and attaching storage

### Network File Systems
- NFS, iSCSI, Ceph, GlusterFS
- Shared storage across multiple nodes
- Good for read-write-many scenarios

## Storage Best Practices

1. **Use Persistent Volumes for Important Data**
   - Never rely on container storage for important data
   - Use PVs for database files, logs, and user uploads

2. **Choose the Right Access Mode**
   - Single node read-write: ReadWriteOnce
   - Multiple nodes read-only: ReadOnlyMany
   - Multiple nodes read-write: ReadWriteMany

3. **Consider Performance Requirements**
   - Local storage (hostPath) is fastest but least flexible
   - Network storage offers better flexibility with some performance trade-off

4. **Plan for Storage Classes**
   - Use StorageClass for dynamic provisioning
   - Define different classes for different performance needs

## Example YAML: Basic Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
  persistentVolumeReclaimPolicy: Retain
```

## Troubleshooting Storage Issues

### Common Issues
- PVC stays in Pending state
- Permission denied errors
- Storage capacity issues
- Node affinity/selector mismatches

### Diagnostic Commands
```bash
# Check PV status
kubectl get pv

# Check PVC status
kubectl get pvc

# Describe PV/PVC for detailed information
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>

# Check pod events for storage-related errors
kubectl describe pod <pod-name>
```

## Next Steps
In the next section, we'll dive into configuring pod volume storage with practical examples.

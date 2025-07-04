# 10.3 Configuring PV Storage

## Introduction to Persistent Volumes (PV)

Persistent Volumes (PVs) are cluster-wide storage resources that exist independently of any pod. They provide a way to manage storage in a way that's decoupled from the pod lifecycle.

## PV Lifecycle

1. **Provisioning**
   - Static: Admin creates PVs manually
   - Dynamic: Created automatically using StorageClass

2. **Binding**
   - PVC requests storage with specific requirements
   - Control loop matches PVC with appropriate PV
   - Binding is exclusive (1:1 relationship)

3. **Using**
   - Pods use PVCs to access the storage
   - Multiple pods can use the same PVC if access mode allows

4. **Releasing**
   - When done, user deletes PVC
   - PV enters "Released" state

5. **Reclaiming**
   - Retain: Manual reclamation
   - Recycle: Basic scrub (deprecated)
   - Delete: Associated storage asset is deleted

## Creating a Persistent Volume

### Basic PV Example (hostPath)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
  persistentVolumeReclaimPolicy: Retain
```

### PV with NFS

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: nfs-server.example.com
    path: "/exports/data"
```

## PV Access Modes

| Access Mode | Description |
|-------------|-------------|
| ReadWriteOnce (RWO) | Read-write by a single node |
| ReadOnlyMany (ROX) | Read-only by many nodes |
| ReadWriteMany (RWX) | Read-write by many nodes |

## PV Reclaim Policies

- **Retain**: Manual reclamation of the resource
- **Recycle**: Basic scrub (rm -rf /volume/*)
- **Delete**: Associated storage asset is deleted

## PV Status

- **Available**: Free and not bound to a claim
- **Bound**: Bound to a claim
- **Released**: Claim has been deleted, but resource not reclaimed
- **Failed**: Automatic reclamation failed

## Capacity and Storage Class

### Setting Capacity
```yaml
spec:
  capacity:
    storage: 10Gi  # Must match underlying storage capacity
```

### Using Storage Classes
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
```

## Node Affinity

Restrict which nodes can use the PV:

```yaml
spec:
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node-1
```

## Mount Options

Specify mount options for the volume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mount-options
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

## Best Practices

1. **Use Meaningful Names**
   - Include storage type and purpose
   - Example: `pv-mysql-data`

2. **Set Appropriate Access Modes**
   - Choose the most restrictive mode that works
   - Prefer ReadWriteOnce when possible

3. **Plan for Capacity**
   - Set realistic storage sizes
   - Monitor usage and scale as needed

4. **Reclaim Policy**
   - Use `Retain` for important data
   - Use `Delete` for temporary storage

5. **Documentation**
   - Document PV purpose and access requirements
   - Include any special mounting instructions

## Troubleshooting

### Common Issues

1. **PV Stays in Pending**
   - Check storage class exists
   - Verify node affinity rules
   - Check capacity requirements

2. **Mount Failures**
   - Verify NFS server is accessible
   - Check permissions on the storage
   - Validate mount options

3. **Capacity Issues**
   - Check actual storage capacity
   - Verify no quota limits are hit

### Diagnostic Commands

```bash
# List all PVs
kubectl get pv

# Get detailed PV information
kubectl describe pv <pv-name>

# Check PV events
kubectl get events --field-selector involvedObject.kind=PersistentVolume

# Check storage classes
kubectl get storageclass
```

## Next Steps
In the next section, we'll explore Persistent Volume Claims (PVCs) and how they interact with PVs to provide storage to pods.

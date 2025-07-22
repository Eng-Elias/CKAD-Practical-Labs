# CKAD Lab 40: Creating a Persistent Volume from a Host Path

## Objective
Create a `hostPath` Persistent Volume (PV) with specific capacity, access modes, and path. This lab reinforces the foundational concepts of provisioning storage in Kubernetes.

## The Challenge
Create a Persistent Volume with the following specifications:
-   **Name:** `pv-foo`
-   **Storage Capacity:** `200Mi`
-   **Access Mode:** `ReadWriteMany` (Allows the volume to be mounted as read-write by many nodes).
-   **Host Path:** `/data/foo2` (A directory on the underlying node).

## Solution Strategy
As with other storage labs, the key is to start with a simple, clean YAML template for the `PersistentVolume` resource. A `hostPath` PV makes a directory from the node's filesystem available to the cluster as a storage resource.

### Step 1: The PV Manifest
Create a YAML file named `pv-hostpath.yaml` with the required specifications.

**`pv-hostpath.yaml`:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-foo
spec:
  # The total storage capacity of this volume
  capacity:
    storage: 200Mi
  
  # How the volume can be mounted. RWX = ReadWriteMany
  accessModes:
    - ReadWriteMany

  # The type of storage. hostPath uses a directory on the node.
  hostPath:
    path: "/data/foo2"
```

**Explanation of Key Fields:**
-   `metadata.name`: The unique name for the PV.
-   `spec.capacity.storage`: The size of the volume.
-   `spec.accessModes`: `ReadWriteMany` (RWX) allows multiple nodes to mount the volume simultaneously. While `hostPath` is tied to a single node, this access mode is still valid to declare.
-   `spec.hostPath.path`: The directory on the node that will be used for storage. **Note:** In a real cluster, the directory specified must exist on the node where the pod consuming the volume is scheduled.

### Step 2: Apply and Verify the PV
Apply the manifest and then use `kubectl describe` to confirm that all the settings are correct.

**Commands:**
```bash
# Apply the manifest to create the PV
kubectl apply -f pv-hostpath.yaml

# Describe the PV to check its details
kubectl describe pv pv-foo
```

**Expected Output for `describe`:**
```
Name:            pv-foo
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        200Mi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/foo2
    HostPathType:
Events:          <none>
```

The `Status: Available` indicates the PV has been successfully provisioned and is ready to be claimed by a Persistent Volume Claim (PVC).

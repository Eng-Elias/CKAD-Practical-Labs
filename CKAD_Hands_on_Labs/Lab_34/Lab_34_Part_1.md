# CKAD Lab 34 (Part 1): Creating a Persistent Volume (PV)

## Objective
This is the first part of a four-part lab covering the end-to-end storage workflow in Kubernetes. In this part, we will create a `hostPath` Persistent Volume (PV), which represents a piece of storage made available to the cluster.

## The Full Challenge (Multi-Part)

1.  **Part 1: Create a Persistent Volume (PV)** with the name `foo-volume`, 2Gi of capacity, `ReadWriteOnce` access mode, a `manual` storage class, and a `hostPath` of `/opt/foo`.
2.  **Part 2: Create a Persistent Volume Claim (PVC)** to request and bind to the PV created in Part 1.
3.  **Part 3: Create a Pod** that mounts and uses the PVC.
4.  **Part 4: Verify** that data written to the volume from within the pod persists on the host node.

## Part 1 Solution: The Persistent Volume

A Persistent Volume is a piece of storage in the cluster that has been provisioned by an administrator. It is a resource in the cluster, just like a CPU or memory.

### Step 1: The PV Manifest
For the CKAD exam, it's best to start with a simple template. The official documentation can be verbose. Here is a clean manifest that meets the lab's requirements.

**`pv.yaml`:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-volume
spec:
  # The total storage capacity of this volume
  capacity:
    storage: 2Gi
  
  # How the volume can be mounted. RWO = ReadWriteOnce (can be mounted by a single node)
  accessModes:
    - ReadWriteOnce

  # The type of storage. For labs, hostPath is common.
  # It uses a directory on the underlying node.
  hostPath:
    path: "/opt/foo"

  # A name that allows a PVC to specifically request this PV.
  # We use 'manual' to prevent dynamic provisioners from interfering.
  storageClassName: manual
```

**Explanation of Key Fields:**
-   `metadata.name`: The unique name of the PV in the cluster.
-   `spec.capacity.storage`: The total size of the volume.
-   `spec.accessModes`: Defines how the volume can be accessed. `ReadWriteOnce` is common for `hostPath` and means it can be mounted as read-write by a single node.
-   `spec.hostPath`: Specifies that this PV uses a directory on the host node's filesystem. **Note:** This is not suitable for production clusters, but is excellent for single-node test environments.
-   `spec.storageClassName`: A label for the PV. A PVC must request the same `storageClassName` to be able to bind to this PV.

### Step 2: Apply the Manifest

**Command:**
```bash
kubectl apply -f pv.yaml
```

### Step 3: Verify the PV
Check the status of the PV. It should be `Available`, waiting for a claim.

**Command:**
```bash
kubectl get pv foo-volume
```

**Expected Output:**
```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
foo-volume   2Gi        RWO            Retain           Available           manual                  10s
```

With the PV created and `Available`, we are now ready to proceed to Part 2, where we will create a Persistent Volume Claim to request this storage.

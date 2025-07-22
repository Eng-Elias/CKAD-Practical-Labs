# CKAD Lab 34 (Part 2): Creating a Persistent Volume Claim (PVC)

## Objective
With a Persistent Volume (PV) now available, the next step is to create a Persistent Volume Claim (PVC). The PVC is a request for storage made by a user or application, which will be fulfilled by our existing PV.

## The Challenge

-   Create a PVC named `foo-pvc`.
-   It must request `400Mi` of storage.
-   It must successfully bind to the `foo-volume` PV created in Part 1.

## Part 2 Solution: The Persistent Volume Claim
A PVC acts as an intermediary between a pod and a PV. It allows developers to request storage without needing to know the underlying details of the storage infrastructure.

### Step 1: The PVC Manifest
The key to binding this PVC to our PV is to ensure the `storageClassName` and `accessModes` match. The requested storage size (`400Mi`) must also be less than or equal to the PV's capacity (`2Gi`).

**`pvc.yaml`:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
spec:
  # This must match the accessModes of the PV
  accessModes:
    - ReadWriteOnce

  # This must match the storageClassName of the PV
  storageClassName: manual
  
  # The amount of storage being requested
  resources:
    requests:
      storage: 400Mi
```

**Explanation of Key Fields:**
-   `metadata.name`: The unique name of the PVC, which we will reference in our pod later.
-   `spec.accessModes`: Must be compatible with the PV's access modes.
-   `spec.storageClassName`: Must exactly match the `storageClassName` of the desired PV. This is the primary mechanism for binding the claim to the volume.
-   `spec.resources.requests.storage`: The amount of storage the application needs.

### Step 2: Apply the Manifest

**Command:**
```bash
kubectl apply -f pvc.yaml
```

### Step 3: Verify the PVC and PV Status
Immediately after creation, the PVC will be in a `Pending` state. The Kubernetes control plane will then find a suitable PV and bind them. Check the status of both the PVC and the PV.

**Commands:**
```bash
# Check the PVC status. It should change from Pending to Bound.
kubectl get pvc foo-pvc

# Check the PV status again. It should now be Bound to our PVC.
kubectl get pv foo-volume
```

**Expected Output for PVC:**
```
NAME      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
foo-pvc   Bound    foo-volume   2Gi        RWO            manual         15s
```

**Expected Output for PV:**
```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
foo-volume   2Gi        RWO            Retain           Bound    default/foo-pvc   manual                  5m
```

The `Bound` status for both resources confirms that the claim was successful. We are now ready for Part 3, where we will create a pod that mounts and uses this PVC.

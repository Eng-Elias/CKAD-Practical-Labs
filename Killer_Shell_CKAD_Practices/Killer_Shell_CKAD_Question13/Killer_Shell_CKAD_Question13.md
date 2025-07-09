# CKAD Practice Question 13: StorageClass and PersistentVolumeClaim

## Question
Team Moonpie, which has the `Namespace` moon, needs more storage. Create a new `PersistentVolumeClaim` named `moon-pvc-126` in that namespace. This claim should use a new `StorageClass` `moon-retain` with the `provisioner` set to `moon-retainer` and the `reclaimPolicy` set to `Retain`. The claim should request storage of `3Gi`, an `accessMode` of `ReadWriteOnce` and should use the new `StorageClass`.

The provisioner `moon-retainer` will be created by another team, so it's expected that the `PVC` will not bind yet. Confirm this by writing the log message from the `PVC` into file `/opt/course/13/pvc-126-reason`.

## Solution

### Step 1: Create the StorageClass
First, create the `StorageClass`. Since the provisioner `moon-retainer` does not exist, any `PVC` using this class will remain in a `Pending` state.

Create a file named `13-sc.yaml`:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: moon-retain
provisioner: moon-retainer
reclaimPolicy: Retain
```

Apply the `StorageClass` manifest:
```bash
kubectl apply -f 13-sc.yaml
```

Verify the `StorageClass` was created:
```bash
kubectl get sc
# NAME          PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
# moon-retain   moon-retainer    Retain          Immediate           false                  5s
```

### Step 2: Create the PersistentVolumeClaim
Next, create the `PersistentVolumeClaim` in the `moon` namespace.

Create a file named `13-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: moon-pvc-126
  namespace: moon
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  storageClassName: moon-retain
```

Apply the `PVC` manifest:
```bash
kubectl apply -f 13-pvc.yaml
```

### Step 3: Verify PVC Status and Capture the Reason
Check the `PVC`'s status. It is expected to be `Pending`.
```bash
kubectl get pvc -n moon
# NAME           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
# moon-pvc-126   Pending                                      moon-retain    15s
```

Now, describe the `PVC` to find the reason it's pending and save it to the specified file.
```bash
# Describe the PVC and extract the event message
kubectl describe pvc moon-pvc-126 -n moon

# The events section will show a message like:
# "waiting for a volume to be created, either by external provisioner \"moon-retainer\" or manually by system administrator"

# Save this message to the file
echo "waiting for a volume to be created, either by external provisioner \"moon-retainer\" or manually by system administrator" > /opt/course/13/pvc-126-reason

# Verify the content of the file
cat /opt/course/13/pvc-126-reason
```

## Explanation

### Key Concepts
1.  **StorageClass**: An object that defines a "class" of storage. It specifies the `provisioner`, `reclaimPolicy`, and other parameters. `StorageClasses` are cluster-scoped.
2.  **PersistentVolumeClaim (PVC)**: A request for storage by a user. It consumes `PV` resources and is namespaced.
3.  **Dynamic Provisioning**: When a `PVC` is created, Kubernetes can automatically provision a `PersistentVolume` (PV) that matches the claim's requirements, based on the `StorageClass`.
4.  **ReclaimPolicy**: Defines what happens to the underlying storage volume after the `PV` it is bound to is released (e.g., `Retain`, `Delete`).
5.  **Pending State**: A `PVC` remains in the `Pending` state until a `PV` that satisfies its request is created and bound to it. In this case, because the specified `provisioner` (`moon-retainer`) doesn't exist, no `PV` will be dynamically provisioned.

### Why This Solution Works
-   The `StorageClass` is created first, defining the storage profile.
-   The `PVC` is then created, requesting storage that conforms to the `StorageClass`.
-   Because the `moon-retainer` provisioner is not running in the cluster, the `PVC` correctly remains in the `Pending` state.
-   `kubectl describe` provides detailed information about the resource, including events, which explain why the `PVC` is pending. This output is then saved to fulfill the final requirement.

### Exam Tips
1.  **Know your Primitives**: Understand the relationship between `StorageClass`, `PVC`, and `PV`.
2.  **Check the Events**: When a resource is not behaving as expected (e.g., a pod is not starting, a `PVC` is pending), `kubectl describe` is your best friend. The `Events` section at the bottom often tells you exactly what's wrong.
3.  **Namespaces Matter**: Remember that `PVCs` are namespaced, while `StorageClasses` and `PVs` are not.
4.  **Imperative vs. Declarative**: While you can create resources imperatively, using declarative YAML files is a more robust and common practice in real-world scenarios and exams.

### Common Mistakes to Avoid
-   Forgetting to specify the `namespace` for the `PVC`.
-   Mismatch between the `storageClassName` in the `PVC` and the name of the `StorageClass`.
-   Incorrectly specifying the `accessModes` or storage request format.
-   Confusing the roles of `provisioner` and `reclaimPolicy`.

## Related Commands
```bash
# List all storage classes
kubectl get storageclass

# Describe a storage class
kubectl describe sc moon-retain

# List all PVCs in a specific namespace
kubectl get pvc -n moon

# Describe a PVC
kubectl describe pvc moon-pvc-126 -n moon
```

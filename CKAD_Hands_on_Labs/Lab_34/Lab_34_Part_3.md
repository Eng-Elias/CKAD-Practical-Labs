# CKAD Lab 34 (Part 3): Mounting a Persistent Volume Claim in a Pod

## Objective
With a bound Persistent Volume Claim (PVC) ready, the final step is to create a pod that mounts this claim, making the persistent storage available to an application.

## The Challenge

-   Create a pod named `foo-pod`.
-   The pod should use the `nginx:alpine` image.
-   Mount the PVC named `foo-pvc` into the pod's container.
-   The mount path inside the container should be `/var/foo`.

## Part 3 Solution: The Pod Manifest
To connect a pod to a PVC, you need to configure two distinct sections in the pod's manifest:

1.  **`spec.volumes`**: This defines a volume that is available to the entire pod. You give it a name and specify its source, which in our case is a `persistentVolumeClaim`.
2.  **`spec.containers.volumeMounts`**: This goes inside a container's definition and tells the container to mount one of the pod's defined volumes into its own filesystem at a specific path.

### Step 1: Build the Pod Manifest
We can generate a basic pod YAML using `kubectl run` with `--dry-run=client -o yaml` and then add the volume configuration.

**`pod.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  # 1. Define a volume for the pod, referencing our PVC.
  volumes:
    - name: foo-storage # A name for this volume, local to the pod definition.
      persistentVolumeClaim:
        claimName: foo-pvc # This must match the name of our PVC.

  containers:
    - name: nginx-container
      image: nginx:alpine
      ports:
        - containerPort: 80
      # 2. Mount the pod's volume into the container.
      volumeMounts:
        - mountPath: "/var/foo" # The path inside the container.
          name: foo-storage      # This must match the volume name from spec.volumes.
```

### Step 2: Apply the Manifest

**Command:**
```bash
kubectl apply -f pod.yaml
```

### Step 3: Verify the Pod and Mounts
Check that the pod is running and then describe it to confirm that the volume was mounted correctly.

**Commands:**
```bash
# Check that the pod is running successfully
kubectl get pod foo-pod

# Describe the pod and look for the 'Volumes' and 'Mounts' sections
kubectl describe pod foo-pod
```

In the output of `kubectl describe`, you should see a `Volumes` section that shows `foo-storage` is of type `PersistentVolumeClaim` and points to `foo-pvc`. You should also see a `Mounts` section inside the container definition, showing that `/var/foo` is mounted from `foo-storage`.

With the pod running and the volume mounted, we are now ready for the final part of the lab: verifying that data written to `/var/foo` persists on the node's host path.

# CKAD Lab 55 (Part 2): Adding the Command and Secret Volume

## Objective
Modify the basic pod manifest generated in Part 1 to include a custom command and mount the `foo-secret` as a volume inside the container.

## Part 2 Solution: Modifying the Pod Manifest
We will now edit the `pod.yaml` file to add the required specifications.

### Step 1: Add the `command` and `args`
To make the container sleep, we override its default command. This is done by adding the `command` and `args` fields to the container definition.

### Step 2: Mount the Secret (A Two-Step Process)
Mounting a secret (or a `ConfigMap`) as a volume always involves two distinct sections in the pod manifest:

1.  **`spec.volumes`**: This is a pod-level field. You define a volume and give it a name. You specify that this volume's source is a `secret` and provide the name of the secret (`foo-secret`).
2.  **`spec.containers.volumeMounts`**: This is a container-level field. You tell the container to mount the volume you just defined (referencing it by name) into a specific `mountPath` within the container's filesystem.

### The Modified `pod.yaml` (Before Affinity)
Here is the updated manifest incorporating the command and the volume mount.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
  namespace: foo-ns
spec:
  containers:
  - name: foo-container # Renamed container
    image: busybox
    # Added command to make the container sleep
    command: ["sleep"]
    args: ["3600"]
    # Step 2: Mount the volume into the container
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/foo"
      readOnly: true
  # Step 1: Define the volume at the pod level
  volumes:
  - name: secret-volume
    secret:
      secretName: foo-secret
```

### Step 3: Prepare for Node Affinity
The final requirement is to schedule this pod only on `node01`. To do this with `nodeAffinity`, we first need to apply a label to `node01` that our pod's affinity rule can match.

**Command:**
```bash
# Apply a unique label to node01
kubectl label nodes node01 disktype=ssd
```

## Next Steps
With the pod manifest now including the command and the secret volume, the final step is to add the `nodeAffinity` section to the spec. Part 3 will cover how to use `kubectl explain` to find the correct YAML structure for the affinity rule and apply the final manifest.

# 15.6 Using Sidecars

This section provides a solution for the sample exam assignment on creating a multi-container pod with a sidecar pattern.

## Task

1.  Create a multi-container pod named `sidecar-pod` in the `ckad-ns3` namespace.
2.  The primary container should use the `busybox` image and write the output of the `date` command to `/var/log/date.log` every five seconds.
3.  A second container should run as a sidecar, using the `nginx` image to provide web access to the log file.
4.  The two containers should communicate using a shared `hostPath` volume.
5.  Ensure the `nginx` container's image is only pulled if it is not already present on the local system.

---

## Solution Walkthrough

This is a more complex task that involves finding documentation, generating YAML, and careful modification.

### 1. Find Documentation (Exam Strategy)

For complex patterns like sidecars, the official Kubernetes documentation is your best resource. A good approach is to search for "sidecar" or "shared volume between containers."

This will lead you to the page **"Communicate Between Containers in the Same Pod Using a Shared Volume,"** which contains a perfect example to start from.

### 2. Generate and Modify YAML

It's best to work with a YAML file for this task.

#### A. Start with a Template

Copy the two-container pod example from the documentation into a new file (e.g., `task15-6.yaml`).

#### B. Generate the `busybox` Command

Getting the command and arguments syntax right in YAML can be tricky. A great shortcut is to use `kubectl run` with `--dry-run` to generate the correct YAML structure for the `busybox` container's looping command.

```bash
kubectl run busybox --image=busybox --dry-run=client -o yaml -- sh -c 'while true; do date >> /var/log/date.log; sleep 5; done'
```

This command will output the YAML for a pod, from which you can copy the `command` and `args` section.

#### C. Assemble the Final YAML

Now, modify the template YAML with the specific requirements:

*   Set `metadata.name` to `sidecar-pod` and `metadata.namespace` to `ckad-ns3`.
*   Change `spec.volumes` from `emptyDir` to `hostPath`.
*   Configure the two containers.

Here is the complete, correct YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
  namespace: ckad-ns3
spec:
  volumes:
  - name: shared-logs
    hostPath:
      # The path on the host is not specified, so we can use a simple one.
      path: /tmp/pod-logs
      type: DirectoryOrCreate

  containers:
  # The primary container writing logs
  - name: busybox-container
    image: busybox
    command: ["sh", "-c"]
    args: ["while true; do date >> /var/log/date.log; sleep 5; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # The sidecar container serving the logs
  - name: nginx-sidecar
    image: nginx
    imagePullPolicy: IfNotPresent # Requirement: only pull if not present
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /usr/share/nginx/html
```

### 3. Debugging YAML Indentation

A common mistake is incorrect indentation. The transcription highlights an error where `args` was accidentally made a child of `volumeMounts`.

**Incorrect:**
```yaml
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
      args: ["..."] # Wrong indentation
```

**Correct:**
```yaml
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
    args: ["..."] # Correct indentation, child of container
```
This is a key point to watch for when editing YAML manually.

### 4. Create Namespace and Pod

Before creating the pod, you must create the namespace if it doesn't exist.

```bash
kubectl create ns ckad-ns3
kubectl create -f task15-6.yaml
```

### 5. Verify the Solution

Check that the pod is running with both containers ready (`2/2`).

```bash
kubectl get pods -n ckad-ns3
```

Finally, `exec` into the `nginx` sidecar and use `cat` to verify that it can access the log file being written by the `busybox` container.

```bash
# Exec into the sidecar
kubectl exec -it sidecar-pod -n ckad-ns3 -c nginx-sidecar -- /bin/bash

# Inside the container, check for the log file
cat /usr/share/nginx/html/date.log
```

This confirms that the shared volume is working correctly and the sidecar pattern is successfully implemented.
This confirms that the shared volume is working correctly and the sidecar pattern is successfully implemented.

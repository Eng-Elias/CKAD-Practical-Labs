# CKAD Lab 34 (Part 4): Verifying and Troubleshooting Persistent Storage

## Objective
In this final part of the lab, we will verify that the storage is truly persistent by writing data from within the pod and checking for it on the host node. We will also cover how to troubleshoot common storage-related errors.

## The Challenge

-   Verify that a file created in the pod's `/var/foo` mount path appears in the node's `/opt/foo` directory.
-   Understand how to diagnose pod startup failures related to volumes.

## Part 4 Solution: Verification and Troubleshooting

### Step 1: The Complete Pod Manifest (Recap)
Here is the final, correct manifest for the pod that mounts the PVC.

**`pod.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  volumes:
    - name: foo-storage
      persistentVolumeClaim:
        claimName: foo-pvc

  containers:
    - name: nginx-container
      image: nginx:alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: "/var/foo"
          name: foo-storage
```

### Step 2: Write Data to the Volume
Once the pod is running, use `kubectl exec` to get a shell inside the container and create a test file in the mounted directory.

**Commands:**
```bash
# Get an interactive shell into the running pod
kubectl exec -it foo-pod -- sh

# Inside the pod's shell, write a file to the mounted volume
/ # echo 'Persistence is working!' > /var/foo/test.txt

# Exit the pod's shell
/ # exit
```

### Step 3: Verify Persistence on the Host Node
To confirm that the data was written to the `hostPath` volume, you would need to access the filesystem of the Kubernetes node where the pod is scheduled. In a real cluster, you would `ssh` into the node.

**Command (to be run on the Kubernetes node):**
```bash
# Check the contents of the hostPath directory
cat /opt/foo/test.txt
```

**Expected Output:**
```
Persistence is working!
```
Seeing this output confirms the entire PV -> PVC -> Pod workflow is functioning correctly.

## Troubleshooting: `CreateContainerError`
In the video, the pod fails to start with a `CreateContainerError`. This is a common issue when working with `hostPath` volumes.

**How to Diagnose:**
1.  **Check Pod Status:** `kubectl get pod foo-pod` shows the `CreateContainerError` status.
2.  **Describe the Pod:** The most important command for troubleshooting pod failures is `kubectl describe pod <pod-name>`.

    ```bash
    kubectl describe pod foo-pod
    ```
3.  **Look at the Events:** At the bottom of the `describe` output, the `Events` section will provide the reason for the failure. In this case, it would show a message similar to:
    `Warning  Failed  ...  Error: failed to start container "nginx-container": Error response from daemon: error while creating mount source path '/opt/foo': mkdir /opt/foo: permission denied`

This error message clearly indicates that the kubelet on the node did not have permission to create or write to the `/opt/foo` directory. In a real exam scenario, the paths provided will be valid and writable. However, knowing how to use `kubectl describe` to find these event messages is a critical skill.

This concludes the four-part lab on Kubernetes storage.

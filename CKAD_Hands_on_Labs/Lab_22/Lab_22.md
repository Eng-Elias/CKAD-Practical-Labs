# CKAD Lab 22: Running a Pod with Two Containers

## Objective
Learn how to create a multi-container pod by defining more than one container in the pod's YAML manifest. This is a common pattern for running a primary application alongside a "sidecar" container for tasks like logging, monitoring, or proxying.

## Key Concept: The `containers` Array
A Pod's specification (`spec`) contains a `containers` field, which is a YAML array. Each item in this array is a container definition, complete with its own name, image, ports, volume mounts, etc.

To create a multi-container pod, you simply add more than one container definition to this array.

## Solution
This lab demonstrates how to generate a manifest for a single-container pod and then edit it to add a second container.

### Step 1: Generate a Base Pod Manifest
First, use `kubectl run` with `--dry-run=client -o yaml` to generate the YAML for a standard `nginx` pod. This saves you from writing it from scratch.

**Command:**
```bash
kubectl run multi-pod --image=nginx --dry-run=client -o yaml > multi-pod.yaml
```

This creates a file named `multi-pod.yaml`.

### Step 2: Edit the YAML to Add a Second Container
Open the `multi-pod.yaml` file. You will see the `containers` array with one entry for the `nginx` container. To add a second container, simply copy the first container's block and paste it as a new item in the array. Then, modify the `name` and `image` for the second container.

**Important Note:** In a multi-container pod, all containers must be long-running processes. If one container exits (like a `busybox` container with no command), the pod will enter a `CrashLoopBackOff` state. For this lab, we will run a second `nginx` container to ensure both containers keep running.

**Modified `multi-pod.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  # --- First Container ---
  - image: nginx
    name: nginx-container-1
    resources: {}

  # --- Second Container ---
  - image: busybox
    name: busybox-container-2
    # Add a command to keep the busybox container running
    command: ['sh', '-c', 'echo Hello from BusyBox!; sleep 3600']
    resources: {}

  dnsPolicy: ClusterFirst
  restartPolicy: Always
```
In this corrected example, we give the `busybox` container a `sleep` command to ensure it stays running.

### Step 3: Apply the Manifest and Verify
Create the pod from the modified manifest and then use `kubectl get` and `kubectl describe` to verify that it's running with two containers.

**Commands:**
```bash
# Create the pod
kubectl apply -f multi-pod.yaml

# Check the pod status. Note the 'READY' column shows 2/2.
kubectl get pod multi-pod
```

**Expected `get` Output:**
```
NAME        READY   STATUS    RESTARTS   AGE
multi-pod   2/2     Running   0          30s
```

Now, describe the pod to see both containers listed.
```bash
kubectl describe pod multi-pod
```

**Expected `describe` Output (snippet):**
```
...
Containers:
  nginx-container-1:
    Container ID:   ...
    Image:          nginx
    ...
  busybox-container-2:
    Container ID:   ...
    Image:          busybox
    Command:
      sh
      -c
      echo Hello from BusyBox!; sleep 3600
    ...
Events:
  ...
```

## Exam Tip
When creating multi-container pods, remember that the `containers` field is an array. Pay close attention to the YAML indentation. A common mistake is to misalign the second container's definition. Also, be prepared to provide a long-running `command` for utility containers like `busybox`.

# CKAD Lab 3: Getting Logs from Pods and Containers

## Objective
Learn how to retrieve logs from running or crashed pods and specific containers, a critical skill for debugging applications running in Kubernetes.

## Key Commands

The primary command for fetching logs is `kubectl logs`.

### 1. Get Logs from a Pod
To view the standard output (stdout) of the primary container in a pod, simply provide the pod's name.

**Command:**
```bash
# Get logs from a pod named 'my-nginx-pod'
kubectl logs my-nginx-pod
```

### 2. Get Logs from a Specific Container
If a pod has multiple containers, you must specify which container's logs you want to see using the `-c` (or `--container`) flag.

**Command:**
```bash
# Get logs from 'container-2' in 'multi-container-pod'
kubectl logs multi-container-pod -c container-2
```

### 3. Get Logs from a Crashed Container
If a container crashes and restarts, `kubectl logs` will only show the logs from the new, running container. To retrieve logs from the *previous*, terminated container to diagnose the crash, use the `-p` (or `--previous`) flag.

**Command:**
```bash
# Get logs from the previously crashed instance of a container in 'my-buggy-pod'
kubectl logs my-buggy-pod --previous
```

### 4. Stream Logs in Real-Time
To follow the log output in real-time, similar to `tail -f`, use the `-f` (or `--follow`) flag.

**Command:**
```bash
# Stream the logs from 'my-app-pod'
kubectl logs my-app-pod --follow
```

## Explanation

`kubectl logs` is your first and most important tool for application debugging in Kubernetes. 

-   When a pod is not behaving as expected, its logs are the first place to look for error messages or other diagnostic information.
-   The `--previous` flag is essential for troubleshooting `CrashLoopBackOff` errors, as it allows you to see the error message that caused the container to terminate before it was restarted by Kubernetes.
-   For pods with multiple containers (e.g., an application container and a sidecar), specifying the container with `-c` is mandatory.

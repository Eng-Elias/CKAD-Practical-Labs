# CKAD Lab 28: Troubleshooting a Failing Pod (`CrashLoopBackOff`)

## Objective
Learn a systematic process for debugging a pod that fails to start and enters a `CrashLoopBackOff` state. This is one of the most critical skills for the CKAD exam.

## The Scenario
You are given a pod manifest (`pod-broken.yaml`) that, when applied, results in a pod that will not stay running.

**`pod-broken.yaml` (The broken file):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-broken-pod
spec:
  containers:
  - name: my-container
    image: nginx
    # This command is intentionally incorrect
    command: ["echo", "Hello World"]
```

## The Troubleshooting Process

### Step 1: Apply and Observe the Failure
First, apply the manifest and check the pod's status.

**Commands:**
```bash
# Apply the broken manifest
kubectl apply -f pod-broken.yaml

# Check the status (it will likely show CrashLoopBackOff)
kubectl get pods
```

**Expected Output:**
```
NAME            READY   STATUS             RESTARTS   AGE
my-broken-pod   0/1     CrashLoopBackOff   1          15s
```

### Step 2: Use `kubectl describe` to Check Events
`kubectl describe` gives you a high-level overview of what happened. This is always the best first step.

**Command:**
```bash
kubectl describe pod my-broken-pod
```
In the `Events` section, you will see that the image was pulled successfully, but the container keeps failing and restarting, leading to the `Back-off restarting failed container` message.

### Step 3: Use `kubectl logs` to Find the Root Cause
Since the container did start (but then failed), it may have produced logs. This is where you'll find the specific error.

**Command:**
```bash
kubectl logs my-broken-pod
```

**Error Output:**
```
exec: "echo": executable file not found in $PATH
```
*(Note: The exact error can vary slightly, like `cannot open echo` as seen in the video, but the meaning is the same.)*

## The Diagnosis and Solution

The error `executable file not found` means the container tried to run a program named `echo` as its main process, but no such file exists in the `nginx` container's PATH. The `nginx` image's default entrypoint is the NGINX web server, not a shell.

To fix this, you must explicitly tell the container to run a shell, and then pass the `echo` command to that shell.

**The Fix:**
The correct way to run a shell command is to use the `command` and `args` fields together.

**`pod-fixed.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-fixed-pod
spec:
  containers:
  - name: my-container
    image: busybox # Using busybox is better as it's designed for shell commands
    # Correct way to run a shell command:
    command: ["/bin/sh"]
    args: ["-c", "echo Hello World && sleep 3600"]
  restartPolicy: OnFailure
```

**Key Changes:**
1.  **Image:** Switched to `busybox`, which is a small image containing common command-line utilities.
2.  **`command`:** Set to `["/bin/sh"]` to start the shell.
3.  **`args`:** Set to `["-c", "..."]` to pass the actual command string to the shell.
4.  **`sleep`:** Added `sleep 3600` because a pod with a command that exits immediately will be marked as `Completed`. To keep it running for observation, a long-lived process is needed.

### Step 4: Clean Up and Re-apply

```bash
# Delete the broken pod
kubectl delete pod my-broken-pod

# Apply the fixed manifest
kubectl apply -f pod-fixed.yaml

# Verify it is now running
kubectl get pod my-fixed-pod
```

## Exam Tip
For the CKAD exam, the `describe` -> `logs` workflow is the fastest way to solve almost any "my pod won't start" problem. Master this process!

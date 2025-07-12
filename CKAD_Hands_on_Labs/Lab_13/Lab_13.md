# CKAD Lab 13: Troubleshooting Pod Failures with `kubectl describe`

## Objective
Learn how to use the `kubectl describe` command to investigate why a pod is failing to start, stuck in a pending state, or in a `CrashLoopBackOff` or `Error` state.

## The Scenario
You've applied a YAML manifest to create a pod, but when you check its status with `kubectl get pods`, you see something other than `Running`. It might be `Pending`, `Error`, or `CrashLoopBackOff`.

**Example of a failing pod:**
```bash
$ kubectl get pods
NAME      READY   STATUS             RESTARTS   AGE
my-pod    0/1     CrashLoopBackOff   4          2m
```
This tells you *that* the pod is failing, but not *why*. To find the reason, you need to dig deeper.

## Solution: Use `kubectl describe`
The `kubectl describe pod <pod-name>` command provides a detailed, human-readable summary of a resource's state, including its configuration, status, and, most importantly, a log of recent events related to it.

### Step 1: Describe the Failing Pod
Run the `describe` command on the pod that is not in a `Running` state.

**Command:**
```bash
kubectl describe pod my-pod
```

### Step 2: Analyze the `Events` Section
Scroll to the very bottom of the output to find the `Events` section. This is the most crucial part for debugging startup issues. It shows a chronological list of actions the Kubernetes control plane took regarding this pod.

**Example `Events` Section:**
```
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  2m                 default-scheduler  Successfully assigned default/my-pod to gke-worker-1
  Normal   Pulling    2m                 kubelet            Pulling image "nginx:latest"
  Normal   Pulled     1m58s              kubelet            Successfully pulled image "nginx:latest"
  Normal   Created    1m58s              kubelet            Created container my-container
  Warning  Failed     1m57s              kubelet            Error: failed to start container "my-container": ...
  Warning  BackOff    1m30s (x5 over 2m) kubelet            Back-off restarting failed container
```

### How to Interpret the Events:
1.  **`Scheduled`**: The scheduler successfully assigned the pod to a node. If a pod is stuck in `Pending`, the error might be here (e.g., no node has sufficient resources).
2.  **`Pulling` / `Pulled`**: The kubelet on the node is attempting to pull the container image. An `ImagePullBackOff` error would show up here if the image name is wrong or the registry is inaccessible.
3.  **`Created`**: The kubelet successfully created the container.
4.  **`Started` / `Failed` / `BackOff`**: This is where application-level errors appear. In the example above, the `Warning` event with the `BackOff` reason indicates the container started but then immediately crashed. The preceding `Failed` event would contain the specific error message from the container runtime.

## Explanation
`kubectl describe` is your primary tool for diagnosing resource issues. While `kubectl logs` is essential for application-level debugging *after* a container is running, `kubectl describe` is for control-plane-level debugging—it tells you what Kubernetes itself is doing and seeing. When a pod fails to start, the `Events` section is almost always the first place you should look to find the root cause.

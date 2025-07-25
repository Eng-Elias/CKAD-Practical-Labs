# CKAD Lab 47 (Part 2): Building and Troubleshooting a Multi-Container Pod

## Objective
In this part, we will edit the YAML template generated in Part 1 to create a pod with two distinct containers, each with its own configuration. We will also troubleshoot a common scheduling issue that can arise in a test environment.

## The Challenge
Complete the `foo-pod` manifest with the following specifications:

-   **Container 1:**
    -   Name: `nginx-container`
    -   Image: `nginx`
    -   Environment Variable: `MY_VAR=foo`
-   **Container 2:**
    -   Name: `busybox-container`
    -   Image: `busybox`
    -   Command: `sleep 3600`
    -   Environment Variable: `MY_VAR=bar`

## Part 2 Solution: The Final Manifest
The `spec.containers` field in a pod manifest is an array, allowing you to define multiple container configurations.

### Step 1: The Completed `pod-multi.yaml`
We edit the template from Part 1 to add the `env` block to the first container and add a second item to the `containers` array for the `busybox` container.

**`pod-multi.yaml` (Final):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  containers:
  # --- Container 1 Definition ---
  - name: nginx-container
    image: nginx
    env:
    - name: "MY_VAR"
      value: "foo"

  # --- Container 2 Definition ---
  - name: busybox-container
    image: busybox
    # Override the container's default entrypoint and command
    command: ["sleep"]
    args: ["3600"]
    env:
    - name: "MY_VAR"
      value: "bar"
```

### Step 2: Apply the Manifest
Apply the final manifest to create the pod.

```bash
kubectl apply -f pod-multi.yaml
```

### Step 3: Bonus Troubleshooting - The `Pending` Pod
After applying the manifest, you might see the pod is stuck in the `Pending` state:

```bash
kubectl get pod foo-pod
# NAME      READY   STATUS    RESTARTS   AGE
# foo-pod   0/2     Pending   0          15s
```

Running `kubectl describe pod foo-pod` would show an event like:
`Warning  FailedScheduling  ... 0/1 nodes are available: 1 node(s) had untolereated taint {foo: bar}`.

**Cause:**
This error means the only available node has a `Taint` (from Lab 49) that this pod does not `Tolerate`. The pod manifest is **syntactically correct**, but the scheduler cannot find a suitable node to place it on.

**Solution:**
To fix this, you would either remove the taint from the node (`kubectl taint nodes <node-name> foo-`) or add the appropriate `tolerations` block to the pod's spec.

## Exam Tip
This lab highlights two key concepts:
1.  The `spec.containers` field is an array. To add more containers, you simply add more items (starting with `- name: ...`) to the list.
2.  A pod stuck in `Pending` is almost always a scheduling issue. Use `kubectl describe pod <pod-name>` to find out why. Common causes are insufficient resources (CPU/memory), or taints and tolerations mismatches.

# CKAD Lab 45 (Part 2): Troubleshooting and Verifying a `livenessProbe`

## Objective
In this part, we will diagnose and resolve the `kubectl apply` error from Part 1, successfully create the pod with its `livenessProbe`, and understand how to verify its state.

## The Error from Part 1: `Forbidden`
When we tried to apply the manifest in Part 1, we received an error similar to this:

`Forbidden: pod updates may not change fields other than ...`

**Explanation:**
This error is fundamental to understanding Kubernetes object management. It occurs when you use `kubectl apply` with a manifest for a resource that **already exists**, and the new manifest attempts to change a field that is **immutable** (cannot be changed) on the running resource. 

For `Pods`, most of the `spec` is immutable after creation, including the image, commands, and probes. Kubernetes interpreted our `apply` command as an attempt to *update* the existing `foo-pod`, which is not allowed.

## Part 2 Solution: The Delete and Re-Apply Pattern
The correct workflow when a resource with the same name exists and you need to change its immutable fields is to delete the old one and create the new one.

### Step 1: Delete the Existing Pod
First, remove the conflicting pod from the cluster.

**Command:**
```bash
kubectl delete pod foo-pod
```

### Step 2: Apply the Correct Manifest
The YAML manifest from Part 1 was correct. The problem was the conflicting pod, not the manifest itself.

**`pod-liveness.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  containers:
  - name: nginx-liveness
    image: nginx
    livenessProbe:
      exec:
        command:
        - "ping"
        - "-c"
        - "2"
        - "8.8.8.8"
      initialDelaySeconds: 20
      periodSeconds: 30
```

### Step 3: Apply and Verify
With the old pod gone, the `apply` command will now create a new pod.

**Commands:**
```bash
# Apply the manifest to create the new pod
kubectl apply -f pod-liveness.yaml

# Check the pod's status
kubectl get pod foo-pod

# Describe the pod to see the probe configuration and events
kubectl describe pod foo-pod
```

**Verification:**
-   `kubectl get pod` will show the pod transitioning to the `Running` state.
-   `kubectl describe pod` will show the `livenessProbe` details. If the `ping` command were to fail consistently, you would see the `RESTARTS` count increase in the `get pod` output, and the `Events` section of the `describe` output would show the container being killed due to liveness probe failure.

## Exam Tip
The `Forbidden: pod updates may not change fields` error is a strong signal that you need to delete and re-create a resource. While `Deployments` are designed to be updated live, for standalone `Pods`, this error means you must delete the old one before applying the new one.

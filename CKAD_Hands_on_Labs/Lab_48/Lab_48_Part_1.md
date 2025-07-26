# CKAD Lab 48 (Part 1): Untainting a Node and Creating a Deployment

## Objective
This two-part lab covers a realistic workflow: untainting a node to make it available for general scheduling, creating a deployment, and then exposing it. This part focuses on the first two steps.

## The Challenge
1.  Ensure a node is available for scheduling by removing a `Taint`.
2.  Create a `Deployment` with the following specifications:
    -   Name: `foo-dp`
    -   Image: `nginx`
    -   Replicas: `3`
    -   Label: `app=my-app` (to be added later)

## Part 1 Solution: Troubleshooting and Creation
This scenario often occurs in test environments where nodes might have been tainted for previous exercises.

### Step 1: Diagnose the Scheduling Problem
First, we attempt to create the deployment.

**Command:**
```bash
kubectl create deployment foo-dp --image=nginx --replicas=3
```

After running this, we check the status and notice the pods are not becoming ready:
```bash
kubectl get deployment foo-dp
# NAME     READY   UP-TO-DATE   AVAILABLE   AGE
# foo-dp   0/3     3            0           20s
```
This indicates a scheduling failure. A quick check with `kubectl describe deployment foo-dp` or `kubectl describe replicaset <replicaset-name>` would show events related to failed scheduling due to taints.

### Step 2: Remove the Taint from the Node
To fix this, we must remove the taint. The `kubectl taint` command is used for both adding and removing taints. To remove a taint, you specify the key and effect of the taint to be removed, followed by a hyphen (`-`).

**Command:**
```bash
# First, identify the taint on the node
kubectl describe node <node-name> | grep -i taints
# Taints:             foo=bar:NoSchedule

# Now, remove the taint
kubectl taint nodes <node-name> foo:NoSchedule-
```

**Note:** The command removes the taint identified by the key (`foo`) and effect (`NoSchedule`). The value (`bar`) is not needed for removal.

### Step 3: Verify the Deployment
Once the taint is removed, the Kubernetes scheduler will automatically notice that the node is now available and will schedule the pending pods from the deployment.

**Verification:**
```bash
# Wait a few moments and check the deployment again
kubectl get deployment foo-dp

# NAME     READY   UP-TO-DATE   AVAILABLE   AGE
# foo-dp   3/3     3            3           2m
```

The `READY` column now shows `3/3`, confirming that the pods have been successfully scheduled and are running.

## Next Steps
With the deployment running, we are now ready for Part 2, where we will:
1.  Add the `app=my-app` label to the deployment.
2.  Expose the deployment as a `NodePort` service.

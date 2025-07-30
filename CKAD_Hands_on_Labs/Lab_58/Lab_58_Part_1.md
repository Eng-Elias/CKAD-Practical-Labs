# CKAD Lab 58 (Part 1): Troubleshooting a Broken Deployment - The Scheduling Problem

## Objective
This is a multi-part troubleshooting lab. Given a non-functional `Deployment`, the goal is to identify the root cause of the failure and apply the necessary fixes.

## The Scenario
A `Deployment` named `foo-redis` exists in the cluster, but none of its pods are running.

```bash
kubectl get deployment
# NAME        READY   UP-TO-DATE   AVAILABLE   AGE
# foo-redis   0/3     3            0           1m
```

## Part 1 Solution: Investigating the `Pending` Pods

### Step 1: Troubleshoot the Pods, Not the Deployment
The first rule of troubleshooting a `Deployment` is to investigate the `Pods` it manages. The `Deployment` controller's job is to create a `ReplicaSet`, which in turn creates `Pods`. The problem almost always lies with the pods.

```bash
kubectl get pods
# NAME                         READY   STATUS    RESTARTS   AGE
# foo-redis-5f8c7f6c9c-abcde   0/1     Pending   0          2m
# foo-redis-5f8c7f6c9c-fghij   0/1     Pending   0          2m
# foo-redis-5f8c7f6c9c-klmno   0/1     Pending   0          2m
```
The `Pending` status is our first major clue. It means the pod has been accepted by the API server, but it cannot be scheduled onto a node.

### Step 2: Describe the Pod to Find the Cause
We use `kubectl describe` on one of the pending pods and look at the `Events` section at the bottom.

```bash
kubectl describe pod foo-redis-5f8c7f6c9c-abcde
```

**Events:**
```
Type     Reason            Age   From               Message
----     ------            ----  ----               -------
Warning  FailedScheduling  2m    default-scheduler  0/1 nodes are available: 1 node(s) didn't match pod node selector.
```

The event message is explicit: the pod has a `nodeSelector` that does not match any available nodes.

### Step 3: Verify the `nodeSelector` and Node Labels
We inspect the `Deployment` to see what `nodeSelector` it's trying to use, and then check the labels on our nodes.

```bash
# Find the nodeSelector in the deployment's pod template
kubectl describe deployment foo-redis
# ...look for Node-Selector: app=foo

# Check the labels on the available nodes
kubectl get nodes --show-labels
# ...confirm that no node has the label 'app=foo'
```

### Step 4: Apply the Fix
The obvious fix is to apply the required label to the node.

```bash
kubectl label node node01 app=foo
```

## The Cliffhanger
Even after applying the correct label, the pods remain in the `Pending` state. This indicates that there is a second, different problem preventing the pods from starting. Part 2 will investigate this new mystery.

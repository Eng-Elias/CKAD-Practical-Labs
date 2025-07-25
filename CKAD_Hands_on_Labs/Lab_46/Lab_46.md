# CKAD Lab 46: Taints and Tolerations

## Objective
Understand and implement `Taints` and `Tolerations` to control pod scheduling. Taints are applied to nodes to repel pods, while tolerations are applied to pods to allow them to be scheduled on nodes with matching taints.

## The Challenge
1.  Apply a `Taint` to a node (e.g., `minikube` or `node01`) with the following properties:
    -   **Key:** `foo`
    -   **Value:** `bar`
    -   **Effect:** `NoSchedule`
2.  Create a pod that can be scheduled on this tainted node.

## Solution Strategy
This is a two-part process. First, we use an imperative command to taint the node. Second, we create a pod manifest that includes a `tolerations` section matching the taint.

### Step 1: Taint the Node
A taint consists of a key, a value, and an effect. The `NoSchedule` effect means that no new pods will be scheduled on the node unless they have a matching toleration.

**Command:**
```bash
# Replace <node-name> with your actual node name (e.g., minikube)
kubectl taint nodes <node-name> foo=bar:NoSchedule
```

**Command Breakdown:**
-   `kubectl taint nodes <node-name>`: The command to add a taint to a specific node.
-   `foo=bar`: The key-value pair of the taint.
-   `:NoSchedule`: The effect of the taint.

**Verification:**
```bash
kubectl describe node <node-name> | grep -i taints
```

### Step 2: Create a Pod with a Toleration
To allow a pod to be scheduled on the tainted node, we must add a `tolerations` block to its `spec`. This block must match the key, value, and effect of the taint.

**`pod-toleration.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-tolerating
spec:
  containers:
  - name: nginx
    image: nginx
  # This tolerations block allows the pod to be scheduled on the tainted node
  tolerations:
  - key: "foo"
    operator: "Equal"
    value: "bar"
    effect: "NoSchedule"
```

**Explanation of Toleration Fields:**
-   `key`, `value`, `effect`: These must match the taint applied to the node.
-   `operator`: Specifies the relationship between the key and value. `Equal` is the default if a `value` is provided. Another option is `Exists`, which only requires the `key` to be present, regardless of its value.

### Step 3: Apply and Verify
Apply the manifest. If the toleration correctly matches the taint, the pod will be scheduled successfully.

**Commands:**
```bash
kubectl apply -f pod-toleration.yaml

# Check that the pod is running
kubectl get pod nginx-tolerating

# Describe the pod and check the Events section for scheduling information
kubectl describe pod nginx-tolerating
```

## Exam Tip
Taints and Tolerations work together like a lock and key. The taint is the lock on the node, and the toleration is the key on the pod. For the CKAD exam, remember the `kubectl taint` command syntax and the structure of the `tolerations` block in the pod spec. It's a common way to test your understanding of scheduling control.

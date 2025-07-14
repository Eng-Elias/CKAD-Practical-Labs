# CKAD Lab 18: Scheduling a Pod to a Specific Node

## Objective
Learn how to schedule a pod to run on a specific node in a Kubernetes cluster. This is a common requirement in the CKAD exam and is useful for scenarios where you need to control pod placement for performance, hardware, or maintenance reasons.

## Key Concept: `nodeName`

The simplest way to constrain a pod to a specific node is by using the `nodeName` field in the pod's specification (`spec`). When you set this field to the name of a valid node, the Kubernetes scheduler bypasses its normal scheduling logic and assigns the pod directly to that node.

While this method is direct, it's also rigid. A more flexible and recommended approach for production is to use `nodeSelector` or `nodeAffinity`, which are covered in later labs. However, for many exam scenarios, `nodeName` is the quickest and most effective solution.

## Solution

This lab demonstrates how to generate a pod manifest, modify it to include `nodeName`, and verify that the pod is scheduled correctly.

### Step 1: List Nodes and Choose a Target
First, list the available nodes in your cluster to choose a target for your pod.

**Command:**
```bash
kubectl get nodes
```

**Example Output:**
```
NAME      STATUS   ROLES    AGE   VERSION
node-01   Ready    <none>   1d    v1.28.0
node-02   Ready    <none>   1d    v1.28.0
node-03   Ready    <none>   1d    v1.28.0
```
Let's say we want to schedule our pod on `node-02`.

### Step 2: Generate the Pod Manifest
Use `kubectl run` with the `--dry-run=client -o yaml` flags to generate the basic YAML for a pod without creating it.

**Command:**
```bash
kubectl run my-pod --image=nginx --dry-run=client -o yaml > pod-on-node.yaml
```

This creates a file named `pod-on-node.yaml` with the following content:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
spec:
  containers:
  - image: nginx
    name: my-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Step 3: Discover the `nodeName` Field (Optional but good practice)
Use `kubectl explain` to explore the pod specification and find the correct field for assigning a node.

**Command:**
```bash
# Explore the pod spec
kubectl explain pod.spec

# Look for a field related to nodes. You'll find 'nodeName'.
# Get details on nodeName
kubectl explain pod.spec.nodeName
```
This will confirm that `nodeName` is a string that specifies which node the pod should be scheduled on.

### Step 4: Edit the YAML to Add `nodeName`
Now, edit the `pod-on-node.yaml` file and add the `nodeName` field under `spec`.

**Modified `pod-on-node.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
spec:
  # Add this line to target a specific node
  nodeName: node-02
  containers:
  - image: nginx
    name: my-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Step 5: Apply and Verify
Create the pod using the modified manifest and use the `-o wide` flag to verify it's running on the correct node.

**Commands:**
```bash
# Create the pod
kubectl apply -f pod-on-node.yaml

# Verify the pod's location
kubectl get pod my-pod -o wide
```

**Expected Output:**
The output will show that `my-pod` is running on `node-02`.
```
NAME     READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
my-pod   1/1     Running   0          10s   10.244.1.5  node-02   <none>           <none>
```

## Exam Tip
This is a very common exam task. Be comfortable with:
1.  Generating YAML with `--dry-run=client -o yaml`.
2.  Knowing that `spec.nodeName` is the field to use.
3.  Verifying the result with `kubectl get pods -o wide`.

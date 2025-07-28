# CKAD Lab 52: Creating a ReplicaSet from YAML

## Objective
Understand the `ReplicaSet` resource and how to create one using a YAML manifest. While `Deployments` are the modern, recommended way to manage stateless applications, `ReplicaSets` are the underlying component that `Deployments` use to manage pod replicas. Understanding them is crucial for a complete picture of Kubernetes controllers.

## The Challenge
1.  Create a new namespace called `foo-ns`.
2.  In the `foo-ns` namespace, create a `ReplicaSet` named `foo-rs` with the following specifications:
    -   Replicas: `7`
    -   Container Image: `nginx`
    -   The `ReplicaSet` should select and manage pods with the label `foo=bar`.

## Solution: The YAML-Only Approach

### Step 1: Create the Namespace
First, we create the namespace where our resources will live.

```bash
kubectl create namespace foo-ns
```

### Step 2: The `ReplicaSet` Manifest
You cannot use `kubectl create replicaset`. You must start with a YAML file, typically by finding an example in the Kubernetes documentation.

The most important part of a `ReplicaSet` manifest is the connection between the `selector` and the pod `template`.
-   `spec.selector`: Tells the `ReplicaSet` which pods to watch and manage.
-   `spec.template`: Defines the pods that the `ReplicaSet` will create to satisfy the `replicas` count. The labels in this template **must** match the `selector`.

**`replicaset.yaml`:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: foo-rs
  # Ensure the ReplicaSet is created in the correct namespace
  namespace: foo-ns
spec:
  replicas: 7
  # This selector defines which pods this ReplicaSet owns.
  selector:
    matchLabels:
      foo: bar
  # This template is used to create new pods.
  template:
    metadata:
      # The labels here MUST match the selector above.
      labels:
        foo: bar
    spec:
      containers:
      - name: nginx
        image: nginx
```

### Step 3: Apply and Verify
Apply the manifest and check the resources in the specified namespace.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f replicaset.yaml

# Check the ReplicaSet in the 'foo-ns' namespace
kubectl get replicaset -n foo-ns

# Check the pods created and managed by the ReplicaSet
kubectl get pods -n foo-ns
```

**Expected Output:**
You should see one `ReplicaSet` named `foo-rs` with a `DESIRED` and `CURRENT` count of 7, and you should see seven pods running with the label `foo=bar`.

## Exam Tip
Remember this key fact for the CKAD exam: `ReplicaSet` is a YAML-only resource for creation. If a question asks you to create a `ReplicaSet`, your first step is to find an example manifest in the Kubernetes documentation and adapt it. Do not waste time trying `kubectl create replicaset`.

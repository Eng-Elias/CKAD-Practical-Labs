# CKAD Lab 56 (Part 2): Building and Troubleshooting a `nodeAffinity` Manifest

## Objective
Using the fields discovered in Part 1, we will now construct the `nodeAffinity` block in our deployment's YAML. This lab focuses on the practical, and often iterative, process of writing complex YAML and using Kubernetes' own error messages to fix it.

## Part 2 Solution: From Theory to Practice

### Step 1: The First (Incorrect) Draft
A common first attempt is to structure all the discovered fields as a series of nested objects (maps).

**`deployment-affinity-v1.yaml` (Incorrect):**
```yaml
# ... deployment spec ...
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              matchExpressions:
                key: disk
                operator: In
                values:
                - ssd
```

### Step 2: The First Error - `got "map", expected "array"`
Applying this manifest will fail with a clear error message:

`error: error validating "deployment-affinity-v1.yaml": error validating data: invalid type for ... spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms: got "map", expected "array";`

**Analysis:**
-   **Path:** The error points directly to `nodeSelectorTerms`.
-   **Problem:** It says `got "map", expected "array"`. This means we structured `nodeSelectorTerms` as an object (a set of key-value pairs), but Kubernetes requires it to be a list.

**The Fix:** We turn it into a list by adding a dash (`-`) before its first element, `matchExpressions`.

### Step 3: The Second (Incorrect) Draft

**`deployment-affinity-v2.yaml` (Incorrect):**
```yaml
# ...
          requiredDuringSchedulingIgnoredDuringExecution:
            # This is now correctly an array
            nodeSelectorTerms:
            - matchExpressions:
                key: disk
                operator: In
                values:
                - ssd
```

### Step 4: The Second Error
Applying this version will also fail, but the error will point to a deeper field:

`... invalid type for ... nodeSelectorTerms[0].matchExpressions: got "map", expected "array";`

**Analysis:**
-   **Path:** The error now points to `matchExpressions`.
-   **Problem:** The same issue! `matchExpressions` also needs to be a list of rule objects.

### Step 5: The Final, Correct Manifest
We apply the final fix by adding a dash (`-`) before the `key` of the `matchExpressions` item.

**`deployment-affinity-final.yaml` (Correct):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      # The affinity block starts here
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            # 'nodeSelectorTerms' is an array of objects
            nodeSelectorTerms:
            - # The first item in the 'nodeSelectorTerms' array
              # 'matchExpressions' is also an array of objects
              matchExpressions:
              - # The first item in the 'matchExpressions' array
                key: disk
                operator: In
                values:
                - ssd
```

### Step 6: Apply and Verify
```bash
# First, ensure a node has the required label
# kubectl label nodes <your-node-name> disk=ssd

kubectl apply -f deployment-affinity-final.yaml

# Verify the pods are scheduled only on the labeled node
kubectl get pods -o wide
```

## Exam Tip
This lab is perhaps the most important lesson for the CKAD exam. You **will** make YAML syntax errors. Do not panic. Read the error message carefully. It will tell you:
1.  **WHERE** the error is (the full path to the field).
2.  **WHAT** the error is (`got "map", expected "array"` is the most common one).

This tells you exactly where to add or remove a dash (`-`) and fix the indentation.

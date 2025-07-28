# CKAD Lab 51 (Part 2): Building and Troubleshooting the `nodeAffinity` Deployment

## Objective
Translate the `nodeAffinity` structure discovered in Part 1 into a working `Deployment` manifest. This lab emphasizes the practical skill of using the API server's validation errors to correct YAML syntax and structure.

## Part 2 Solution: The "Apply, Read Error, Fix" Cycle
This iterative process is one of the fastest ways to build a correct manifest when you are unsure of the exact structure.

### Step 1: The First Draft (with errors)
We start by assuming all the nested fields under `affinity` are simple objects (maps). We also intentionally use the wrong key (`value` instead of `values`).

**`deployment-affinity-v1.yaml` (Incorrect):**
```yaml
# ... spec.template.spec ...
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              matchExpressions:
                key: foo
                operator: In
                value: # <-- Incorrect key
                - bar
```

### Step 2: The First Error (`nodeSelectorTerms`)
Applying this manifest fails with a familiar error:

`... invalid type for ... nodeSelectorTerms: got "map", expected "array";`

**Fix:** The error tells us `nodeSelectorTerms` must be a list. We add a dash (`-`) before `matchExpressions`.

### Step 3: The Second Error (`matchExpressions`)
After fixing the first error and re-applying, we get a new error pointing deeper into the structure:

`... invalid type for ... nodeSelectorTerms[0].matchExpressions: got "map", expected "array";`

**Fix:** The error tells us `matchExpressions` must also be a list. We add a dash (`-`) before its `key`.

### Step 4: The Third Error (`Unknown field "value"`)
After fixing the array structures and re-applying, we get a different kind of error:

`... error validating data: unknown field "value" in ... matchExpressions[0];`

**Fix:** The error tells us the key `value` is not recognized. We consult `kubectl explain` or our memory and correct it to `values`.

### Step 5: The Final, Correct Manifest
After applying all the fixes, we arrive at the correct manifest.

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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: foo
                operator: In
                values:
                - bar
```

### Step 6: Apply and Verify
```bash
# Ensure the node is labeled
# kubectl label nodes <your-node-name> foo=bar

kubectl apply -f deployment-affinity-final.yaml

# Verify the deployment is running and all pods are on the correct node
kubectl get deployment foo-deployment
kubectl get pods -o wide
```

## Exam Tip
This lab perfectly simulates the pressure of the CKAD exam. You will write imperfect YAML. The key to success is not to write it perfectly the first time, but to quickly and accurately interpret the validation errors from `kubectl apply` to fix your manifest. Trust the error messages—they tell you exactly where to look and what to fix.

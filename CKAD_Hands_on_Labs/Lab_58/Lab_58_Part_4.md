# CKAD Lab 58 (Part 4): The Final Fix and a Working Deployment

## Objective
With the problem isolated to the `ConfigMap` volume, the final step is to correctly re-introduce it into the now-working `Deployment` manifest and verify the complete solution.

## Part 4 Solution: Incremental Re-assembly
The best way to fix the faulty component is to rebuild it incrementally, using a simple, known-good example from the documentation as a base.

### Step 1: Add a Simple `ConfigMap` Volume Definition
We edit the `deployment.yaml` file and add a simple `ConfigMap` volume definition to the `volumes` array. We test this change before proceeding.

```bash
# Delete the running deployment to start fresh
kubectl delete deployment foo-redis

# Edit deployment.yaml, then apply
kubectl apply -f deployment.yaml

# Check that pods still run. This confirms the volume *definition* is valid.
kubectl get pods
```

### Step 2: Add the `volumeMount`
Once the volume definition is confirmed to be working, we add the corresponding `volumeMount` to the container spec.

### The Final, Corrected `deployment.yaml`
This manifest contains all the fixes from the troubleshooting process and results in a working `Deployment`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: foo-container
        image: redis:alpine
        volumeMounts:
        - name: data
          mountPath: /data
        # The correctly added volumeMount
        - name: cm1-volume
          mountPath: /etc/foo
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: foo
                operator: In
                values:
                - bar
      volumes:
      - name: data
        emptyDir: {}
      # The correctly added, simple volume definition
      - name: cm1-volume
        configMap:
          name: cm1
```

### Step 3: Final Verification
We apply the final manifest and see that all pods are now running correctly.

```bash
kubectl delete deployment foo-redis
kubectl apply -f deployment.yaml
kubectl get pods
# NAME                         READY   STATUS    RESTARTS   AGE
# foo-redis-qwert-12345        1/1     Running   0          15s
# ... (all 3 pods running)
```

## Lab Summary
This multi-part lab demonstrated a realistic troubleshooting scenario where multiple, independent problems occurred:
1.  **Initial Problem:** A `nodeSelector` was preventing pods from being scheduled.
2.  **Hidden Problem:** A `nodeAffinity` rule was the *real* scheduling constraint.
3.  **Second Problem:** A required `ConfigMap` was missing, causing volume mount failures.
4.  **Third Problem:** The `ConfigMap` volume definition in the original manifest was overly complex or incorrect, also causing mount failures.

By using `kubectl describe`, inspecting the full YAML, and isolating components, we were able to solve each problem systematically.

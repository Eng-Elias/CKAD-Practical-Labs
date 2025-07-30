# CKAD Lab 58 (Part 3): Isolating the Faulty Component

## Objective
With multiple potential issues (scheduling, volumes, resource requests), the best strategy is to simplify the problem. By removing components one by one, we can isolate the exact source of the failure.

## Part 3 Solution: Simplify, Test, Isolate

### Step 1: Save the Current State and Reset
First, we save the YAML of the broken `Deployment` to a file so we can edit it. Then, we delete the `Deployment` from the cluster to ensure a clean slate.

```bash
# Save the current state to a file
kubectl get deployment foo-redis -o yaml > deployment.yaml

# Delete the broken deployment
kubectl delete deployment foo-redis
```

### Step 2: Simplify the Manifest
Now, we edit the `deployment.yaml` file and remove potential sources of error. A good strategy is to remove the most recently added or most complex parts. In this case, we will comment out the `ConfigMap` volume and its corresponding `volumeMount`.

**`deployment.yaml` (Simplified):**
```yaml
# ... (apiVersion, kind, metadata, etc.)
spec:
  # ... (replicas, selector, template metadata, etc.)
  template:
    spec:
      containers:
      - name: foo-container
        # ... (image, ports, etc.)
        volumeMounts:
        # The ConfigMap mount is temporarily removed
        # - name: cm1
        #   mountPath: /etc/foo
        - name: data
          mountPath: /data
      # ... (affinity, etc.)
      volumes:
      # The ConfigMap volume definition is temporarily removed
      # - name: cm1
      #   configMap:
      #     name: cm1
      - name: data
        emptyDir: {}
```

### Step 3: Re-apply and Observe
We apply the simplified manifest.

```bash
kubectl apply -f deployment.yaml

kubectl get pods
# NAME                         READY   STATUS    RESTARTS   AGE
# foo-redis-asdfg-12345        1/1     Running   0          10s
# foo-redis-asdfg-67890        1/1     Running   0          10s
# foo-redis-asdfg-zxcvb        1/1     Running   0          10s
```

**Success!** The pods are now running. This definitively proves that the `nodeAffinity`, the `emptyDir` volume, and the base `Deployment` configuration are all correct. The problem is isolated to the `ConfigMap` volume definition or mount.

## Next Steps
Now that we know exactly where the problem is, Part 4 will focus on analyzing the `ConfigMap` volume configuration from the original YAML to find and fix the final error.

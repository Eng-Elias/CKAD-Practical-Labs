# CKAD Lab 44 (Part 2): Building and Troubleshooting a `readinessProbe`

## Objective
Using the API structure we discovered with `kubectl explain` in Part 1, we will now construct the final pod manifest, apply it, and troubleshoot a common error related to nested YAML objects.

## The Challenge
Create a pod named `foo-pod` with an `nginx` container. Configure a `readinessProbe` that performs an HTTP GET request on the path `/` on port `80`.

## Part 2 Solution: The Final Manifest
From our investigation in Part 1, we know the structure is `readinessProbe` -> `httpGet` -> (`path` and `port`).

### Step 1: The Common Mistake
A frequent error is to place the `path` and `port` fields directly under `readinessProbe`, forgetting the `httpGet` object that should contain them.

**Incorrect Manifest:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    readinessProbe:
      # INCORRECT: These fields belong inside an httpGet object
      path: "/"
      port: 80
```

Applying this manifest will fail with an error like:
`error: error validating "pod.yaml": error validating data: ValidationError(Pod.spec.containers[0].readinessProbe): unknown field "path" in io.k8s.api.core.v1.Probe;`

This "unknown field" error is a clear sign that you've placed a field at the wrong level of the YAML hierarchy.

### Step 2: The Correct Manifest
The fix is to introduce the `httpGet` object and nest the `path` and `port` fields inside it, as discovered with `kubectl explain`.

**`pod-readiness.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    # The probe checks if the container is ready to receive traffic
    readinessProbe:
      # Correctly nested httpGet object
      httpGet:
        path: "/"
        port: 80
      # Optional: Configure probe timing
      initialDelaySeconds: 5 # Wait 5s before first probe
      periodSeconds: 10      # Probe every 10s
```

### Step 3: Apply and Verify
Apply the correct manifest and use `kubectl describe` to see the probe's status.

**Commands:**
```bash
# Apply the correct manifest
kubectl apply -f pod-readiness.yaml

# Describe the pod to see the probe's configuration and events
kubectl describe pod foo-pod
```

**Verification:**
-   Initially, `kubectl get pod foo-pod` will show `READY: 0/1`.
-   After the `initialDelaySeconds` and the first successful probe, the status will change to `READY: 1/1`.
-   The `Events` section of the `describe` output will show events related to the readiness probe's success or failure.

## Exam Tip
YAML structure is everything. The "unknown field" error is your friend—it tells you that you've made a structural mistake. When you see it, go back to `kubectl explain resource.spec...` and double-check the nesting of the field you're trying to use. For probes, remember the hierarchy: `readinessProbe` -> `httpGet`/`exec`/`tcpSocket` -> specific fields.

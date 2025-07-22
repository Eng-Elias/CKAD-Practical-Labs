# CKAD Lab 31 (Part 1): Creating and Configuring a Deployment

## Objective
This two-part lab covers the complete lifecycle of a Deployment: creating it with a custom rollout strategy, performing a rolling update to a new version, and rolling back to a previous version.

## The Challenge

1.  **Create a Deployment** named `foo-deploy`.
2.  It should have **4 replicas**.
3.  It should use the `nginx:1.16` image.
4.  Configure a **rolling update strategy** where:
    -   `maxSurge` is 25%.
    -   `maxUnavailable` is 25%.
5.  (Part 2) Upgrade the image to `nginx:1.17`.
6.  (Part 2) Roll back the deployment to the original `nginx:1.16` version.

## Part 1 Solution: Creating the Initial Deployment

### Step 1: Understanding the Deployment Strategy
The most common mistake when configuring a rolling update is misplacing the `maxSurge` and `maxUnavailable` fields. These fields belong inside a `rollingUpdate` object, which itself is under the `strategy` block.

Let's use `kubectl explain` to see the correct path:
```bash
# This shows the 'strategy' field
kubectl explain deployment.spec.strategy

# This shows the fields within the 'rollingUpdate' object
kubectl explain deployment.spec.strategy.rollingUpdate
```
This confirms the correct path is `spec.strategy.rollingUpdate.maxSurge`.

### Step 2: Build the Deployment Manifest
Now we can create the `deployment.yaml` file with the correct structure.

**`deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  # This section defines the rollout strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      # During an update, we can have 25% (1 pod) more than the desired replica count.
      maxSurge: 25%
      # During an update, we can have 25% (1 pod) of our pods unavailable.
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
```

**Explanation of Strategy:**
-   **`replicas: 4`**: The desired state is 4 running pods.
-   **`maxSurge: 25%`**: During the update, Kubernetes can create 1 extra pod (`25%` of 4) before terminating an old one. The total number of pods can go up to 5.
-   **`maxUnavailable: 25%`**: During the update, Kubernetes can terminate 1 pod (`25%` of 4) before the new one is ready. The total number of available pods can drop to 3.

This manifest is now correctly configured and ready to be deployed. Part 2 will cover applying this manifest, performing the upgrade, and executing the rollback.

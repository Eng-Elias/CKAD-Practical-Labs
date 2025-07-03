# 15.7 Fixing a Deployment

This section provides a solution for the sample exam assignment on fixing a broken deployment manifest.

## Task

Start a deployment from the `redis.yaml` file provided in the course repository and fix any problems that may occur while starting it.

---

## Solution Walkthrough

This assignment tests your ability to debug and fix a common issue with Kubernetes manifests: deprecated API versions.

### 1. Identify the Problem

The first step is to try and create the resource to see the error.

```bash
kubectl create -f redis.yaml
```

This command will fail with an error message similar to this:

```
error: unable to recognize "redis.yaml": no matches for kind "Deployment" in version "apps/v1beta1"
```

The key part of the error is `no matches for... in version "apps/v1beta1"`. This tells you that the `apiVersion` used in the YAML file is no longer supported by the cluster.

### 2. Find the Correct API Version

To find the currently supported API versions, you can use the `kubectl api-versions` command.

```bash
kubectl api-versions
```

Grepping the output for `apps` will show that `apps/v1` is available, but `apps/v1beta1` is not.

### 3. Fix the YAML File

In this case, the fix is straightforward. Simply edit the `redis.yaml` file and change the `apiVersion`.

**Original (Incorrect):**
```yaml
apiVersion: apps/v1beta1
kind: Deployment
...
```

**Fixed:**
```yaml
apiVersion: apps/v1
kind: Deployment
...
```

### 4. Apply the Fix

After saving the change, run the `create` command again.

```bash
kubectl create -f redis.yaml
```

This time, the deployment will be created successfully.

### Exam Tip

While this example only required changing the `apiVersion`, be aware that sometimes API changes are more complex. Properties within a resource definition can also be renamed or restructured. In those cases, you would need to use `kubectl explain` or refer to the official Kubernetes documentation to find the correct structure for the new API version.

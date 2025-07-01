# 15.15 Managing Pod Permissions

## Assignment
Create a pod manifest file with the following specifications:
- Name: `sleepy-box`
- Image: `busybox:latest`
- Command: `sleep 3600`
- The primary user should be a member of supplementary group ID 2012

## Solution Walkthrough

### 1. Generate the Basic Pod Manifest
First, let's create a basic pod manifest using `kubectl run` with the `--dry-run=client` flag:

```bash
kubectl run sleepy-box --image=busybox --dry-run=client -o yaml -- sleep 3600 > pod-permissions.yaml
```

This will generate a basic pod manifest. Now, let's add the security context for the supplementary group.

### 2. Understanding Security Context
We need to add a `securityContext` to the pod specification to set the supplementary group. The key field we'll use is `fsGroup`, which sets the filesystem group for all containers in the pod.

### 3. Edit the Pod Manifest
Edit the generated `pod-permissions.yaml` file to include the security context:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sleepy-box
spec:
  securityContext:
    fsGroup: 2012  # This sets the supplementary group
  containers:
  - name: sleepy-box
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      allowPrivilegeEscalation: false
```

### 4. Apply the Configuration
```bash
kubectl apply -f pod-permissions.yaml
```

### 5. Verify the Configuration
Check that the pod is running and has the correct security context:

```bash
# Check pod status
kubectl get pod sleepy-box

# Verify the security context
kubectl get pod sleepy-box -o yaml | grep -A 5 securityContext
```

## Common Issues and Solutions

1. **Incorrect fsGroup Format**
   - Error: `unknown field "fsgroup"`
   - Solution: Ensure you use `fsGroup` (with uppercase 'G')

2. **Permission Denied**
   - If the pod fails to start with permission errors, check:
     - The group ID (2012) exists on the node
     - The container user has the necessary permissions

3. **Security Context Not Applied**
   - Verify the pod's YAML to ensure the security context is correctly nested under `spec`
   - Check for typos in the YAML

## Exam Tips

1. **Security Context Fields**
   - `fsGroup`: Sets the filesystem group for all containers
   - `runAsUser`: Sets the user ID for the container process
   - `runAsGroup`: Sets the primary group ID for the container process
   - `supplementalGroups`: Additional groups for the container process

2. **Verification**
   Always verify your configuration by:
   ```bash
   # Check pod details
   kubectl describe pod <pod-name>
   
   # Get pod YAML
   kubectl get pod <pod-name> -o yaml
   ```

3. **Cleanup**
   ```bash
   kubectl delete pod sleepy-box
   ```

## Key Takeaways
- Use `securityContext` to configure pod and container security parameters
- `fsGroup` sets the filesystem group for all containers in the pod
- Always verify security context settings after pod creation
- Be careful with YAML indentation when adding security contexts
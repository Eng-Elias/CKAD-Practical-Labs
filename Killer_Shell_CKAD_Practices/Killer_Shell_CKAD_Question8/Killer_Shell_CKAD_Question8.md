# CKAD Practice Question 8: Debugging and Rolling Back a Deployment

## Question
There is an existing deployment named `api-new-c32` in the `neptune` namespace. A developer made an update to the deployment, but the updated version never came online.

**Tasks**:
1. Check the deployment history and find a working revision
2. Roll back to the working revision
3. Identify and report the error that caused the issue

**Important Notes**:
- The deployment should maintain its desired replica count
- Document the error to prevent it from happening again
- The application is currently experiencing issues, so some downtime is expected

## Solution

### Step 1: Inspect the Deployment and Pods
```bash
# Check the deployment status
kubectl get deployment api-new-c32 -n neptune

# List all pods in the neptune namespace
kubectl get pods -n neptune

# Look for pods with issues (ImagePullBackOff, CrashLoopBackOff, etc.)
kubectl get pods -n neptune | grep -v Running
```

### Step 2: Debug the Failing Pod
```bash
# Describe the failing pod to see error messages
kubectl describe pod <failing-pod-name> -n neptune

# Check the pod logs for any error messages
kubectl logs <failing-pod-name> -n neptune

# Look at the events for the pod
kubectl get events --field-selector involvedObject.name=<failing-pod-name> -n neptune
```

### Step 3: Check Deployment History
```bash
# View the deployment history
kubectl rollout history deployment api-new-c32 -n neptune

# Get details about a specific revision
kubectl rollout history deployment api-new-c32 --revision=<revision-number> -n neptune
```

### Step 4: Roll Back to a Working Revision
```bash
# Roll back to the previous working version
kubectl rollout undo deployment api-new-c32 -n neptune

# Or specify a specific revision to roll back to
kubectl rollout undo deployment api-new-c32 --to-revision=<revision-number> -n neptune

# Verify the rollback was successful
kubectl rollout status deployment api-new-c32 -n neptune
kubectl get pods -n neptune
```

### Step 5: Identify and Report the Issue
Based on the error messages, identify the root cause. In this case:
1. The deployment was updated with an incorrect image name (`nginx:116.3` instead of `nginx:1.16.3`)
2. This caused the pods to fail with an `ImagePullBackOff` error
3. The solution was to roll back to the previous working revision

## Explanation

### Key Concepts
1. **Deployment Rollouts**: Kubernetes tracks changes to deployments as revisions
2. **Rollback**: The process of reverting to a previous working state
3. **Pod Lifecycle**: Understanding pod states like `ImagePullBackOff`, `CrashLoopBackOff`, etc.
4. **Debugging Tools**: Using `kubectl describe` and `kubectl logs` to diagnose issues

### Why This Solution Works
- The deployment history maintains previous revisions of the deployment
- Rolling back to a known good revision quickly restores service
- The `kubectl rollout undo` command simplifies the rollback process
- Proper verification ensures the rollback was successful

### Exam Tips
1. **Check Pod Status**: Always check pod status and events when debugging
2. **Use Describe**: `kubectl describe` provides detailed information about resources
3. **Rollout Commands**: Know how to use `kubectl rollout` commands for managing deployments
4. **Verify Changes**: Always verify that changes have the intended effect
5. **Documentation**: Be familiar with common error messages and their meanings

### Common Mistakes to Avoid
- Not checking pod status after making changes
- Forgetting to specify the namespace with `-n`
- Not verifying that the rollback was successful
- Ignoring error messages in pod descriptions
- Not documenting the root cause of the issue

## Additional Practice
1. Intentionally break a deployment and practice rolling back
2. Set up a deployment with a failing health check and debug it
3. Practice rolling updates with `kubectl set image`
4. Configure resource limits and debug pods that exceed them
5. Practice debugging services that can't reach their pods

## Related Commands
```bash
# Get detailed information about the deployment
kubectl describe deployment api-new-c32 -n neptune

# Watch the rollout status in real-time
kubectl rollout status deployment api-new-c32 -n neptune -w

# View the deployment configuration
kubectl get deployment api-new-c32 -o yaml -n neptune

# Scale the deployment
kubectl scale deployment api-new-c32 --replicas=3 -n neptune

# Pause and resume a rollout
kubectl rollout pause deployment api-new-c32 -n neptune
kubectl rollout resume deployment api-new-c32 -n neptune
```
 
# 7.3 Understanding Deployment Updates

## Introduction to Deployment Updates

Kubernetes Deployments provide powerful mechanisms for updating containerized applications with minimal downtime. This section covers the different update strategies and how to manage them effectively.

## Update Strategies

### 1. Rolling Updates (Default)

Rolling updates gradually replace old pods with new ones, ensuring continuous availability.

**Key Features:**
- Zero-downtime updates
- Controlled by `maxSurge` and `maxUnavailable` parameters
- Default strategy for Deployments

**Example Configuration:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

### 2. Recreate Strategy

Terminates all existing pods before creating new ones, causing a brief service interruption.

**Use Cases:**
- When your application doesn't support multiple versions running simultaneously
- During major version upgrades that require a clean state

**Example Configuration:**
```yaml
spec:
  strategy:
    type: Recreate
```

## Performing Updates

### Updating a Deployment

#### Method 1: Using `kubectl set image`
```bash
kubectl set image deployment/my-app nginx=nginx:1.19.10
```

#### Method 2: Editing the Deployment
```bash
kubectl edit deployment my-app
# Update the container image version and save
```

#### Method 3: Applying an Updated Manifest
```bash
# Make changes to your deployment YAML
kubectl apply -f deployment.yaml
```

### Monitoring the Update

```bash
# Watch the update progress
kubectl rollout status deployment/my-app

# View the rollout history
kubectl rollout history deployment/my-app

# View details of a specific revision
kubectl rollout history deployment/my-app --revision=2
```

## Rollback Strategies

### Rolling Back to a Previous Version

```bash
# View rollout history
kubectl rollout history deployment/my-app

# Rollback to the previous version
kubectl rollout undo deployment/my-app

# Rollback to a specific revision
kubectl rollout undo deployment/my-app --to-revision=2
```

### Pausing and Resuming Rollouts

```bash
# Pause a rollout
kubectl rollout pause deployment/my-app

# Make multiple changes while paused
kubectl set image deployment/my-app nginx=nginx:1.19.10
kubectl set resources deployment/my-app -c=nginx --limits=cpu=200m,memory=512Mi

# Resume the rollout
kubectl rollout resume deployment/my-app
```

## Advanced Update Scenarios

### Blue-Green Deployments

1. Deploy the new version alongside the old one
2. Switch traffic when the new version is ready
3. Remove the old version

```bash
# Create a new deployment with a different label
kubectl create -f new-deployment.yaml

# Update the service to point to the new deployment
kubectl patch svc my-service -p '{"spec":{"selector":{"app":"my-app-v2"}}}'

# Verify traffic is flowing to the new version
# Delete the old deployment when ready
kubectl delete deployment my-app-v1
```

### Canary Deployments

Gradually shift traffic to the new version.

```yaml
# Initial deployment with 90% traffic to v1, 10% to v2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
    version: v1
spec:
  replicas: 9
  # ... rest of the deployment spec
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
    version: v2
spec:
  replicas: 1
  # ... rest of the deployment spec
```

## Best Practices for Updates

1. **Use Readiness Probes**: Ensure pods are ready before receiving traffic
2. **Set Resource Limits**: Prevent resource starvation during updates
3. **Monitor Rollouts**: Use `kubectl rollout status` to monitor progress
4. **Test Rollbacks**: Regularly test your rollback procedures
5. **Use Semantic Versioning**: Follow semantic versioning for container images

## Troubleshooting Updates

### Common Issues

1. **ImagePullBackOff**: Check image name and tags
2. **CrashLoopBackOff**: Check container logs
3. **Update Stuck**: Check resource quotas and node resources
4. **Rollback Failing**: Check if the previous revision is still available

### Useful Commands

```bash
# View detailed deployment status
kubectl describe deployment my-app

# View pod events
kubectl get events --field-selector involvedObject.name=my-app-xxxxx

# View container logs
kubectl logs -l app=my-app
```

## Next Steps

In the next section, we'll explore labels, selectors, and annotations in more detail, which are essential for managing deployments effectively.

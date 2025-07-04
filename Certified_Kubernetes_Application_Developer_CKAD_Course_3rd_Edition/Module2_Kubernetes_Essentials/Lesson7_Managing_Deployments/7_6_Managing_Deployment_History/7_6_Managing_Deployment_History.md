# 7.6 Managing Deployment History

## Introduction to Deployment History

Kubernetes maintains a history of all revisions of a Deployment, allowing you to roll back to any previous version. This section covers how to view, manage, and utilize this history effectively.

## How Deployment History Works

- Each update to a Deployment creates a new revision
- The Deployment controller stores these revisions as ReplicaSets
- By default, Kubernetes keeps 10 revisions (configurable via `revisionHistoryLimit`)

## Viewing Deployment History

### Basic Commands

```bash
# View rollout history
kubectl rollout history deployment/<deployment-name>

# View detailed information about a specific revision
kubectl rollout history deployment/<deployment-name> --revision=<revision-number>
```

### Example Output

```bash
$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/nginx-deployment nginx=nginx:1.19.10
3         kubectl set image deployment/nginx-deployment nginx=nginx:1.20.0
```

## Configuring Revision History

### Setting revisionHistoryLimit

You can control how many old ReplicaSets to retain:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  revisionHistoryLimit: 5  # Keep last 5 revisions
  # ... rest of the deployment spec
```

**Best Practices:**
- Set an appropriate limit based on your needs (default is 10)
- More revisions provide better rollback options but consume more resources
- Consider your update frequency when setting this value

## Rolling Back Deployments

### Basic Rollback

```bash
# Rollback to the previous version
kubectl rollout undo deployment/<deployment-name>

# Rollback to a specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=<revision-number>
```

### Example: Rolling Back to a Specific Revision

1. First, view the history:
   ```bash
   kubectl rollout history deployment/nginx-deployment
   ```
   
2. Then rollback to the desired revision:
   ```bash
   kubectl rollout undo deployment/nginx-deployment --to-revision=2
   ```

3. Verify the rollback:
   ```bash
   kubectl get deployment nginx-deployment -o wide
   kubectl describe deployment nginx-deployment
   ```

## Change-Cause Annotation

### Adding Change Cause

To make your history more meaningful, add a change cause:

```bash
kubectl annotate deployment/nginx-deployment \
    kubernetes.io/change-cause="Update to nginx 1.19.10 for CVE-2021-23017"
```

### Viewing Change Causes

```bash
kubectl rollout history deployment/nginx-deployment
```

## Practical Example: Managing History

### 1. Create a Deployment

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  revisionHistoryLimit: 5
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
        image: nginx:1.18.0
        ports:
        - containerPort: 80
```

### 2. Apply the Deployment

```bash
kubectl apply -f nginx-deployment.yaml
kubectl rollout status deployment/nginx-deployment
```

### 3. Make Some Updates

```bash
# First update
kubectl set image deployment/nginx-deployment nginx=nginx:1.19.10
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Update to nginx 1.19.10"

# Second update
kubectl set image deployment/nginx-deployment nginx=nginx:1.20.0
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Update to nginx 1.20.0"

# Third update
kubectl set image deployment/nginx-deployment nginx=nginx:1.21.0
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Update to nginx 1.21.0"
```

### 4. View History

```bash
kubectl rollout history deployment/nginx-deployment
```

### 5. Rollback to a Previous Version

```bash
# Rollback to nginx 1.19.10 (revision 2)
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

## Advanced Rollback Scenarios

### Rollback When Deployment is Stuck

If a deployment is stuck in progress:

```bash
# Check rollout status
kubectl rollout status deployment/nginx-deployment

# If stuck, you can cancel the rollout
kubectl rollout undo deployment/nginx-deployment
```

### Rollback to a Specific Configuration

To rollback to a specific configuration:

1. Export the revision to a file:
   ```bash
   kubectl get replicaset <replica-set-name> -o yaml > revision.yaml
   ```

2. Edit the file to remove unnecessary fields (status, metadata.uid, etc.)

3. Apply the configuration:
   ```bash
   kubectl apply -f revision.yaml
   ```

## Best Practices

1. **Use Meaningful Change Causes**: Always document why changes were made
2. **Set Reasonable History Limits**: Balance between rollback options and resource usage
3. **Test Rollbacks**: Periodically test rollback procedures
4. **Monitor Resource Usage**: Old ReplicaSets consume cluster resources
5. **Clean Up Old Revisions**: Manually clean up if you need to free resources

## Troubleshooting

### Issue: No Rollout History Available

**Possible Causes**:
- `revisionHistoryLimit` is set to 0
- The deployment was created with `--record=false`
- The deployment was created before enabling history

**Solution**:
1. Check the deployment configuration:
   ```bash
   kubectl get deployment <deployment-name> -o yaml | grep revisionHistoryLimit
   ```
2. If needed, update the deployment:
   ```bash
   kubectl patch deployment <deployment-name> -p '{"spec":{"revisionHistoryLimit": 10}}'
   ```

### Issue: Rollback Fails

**Possible Causes**:
- The revision no longer exists
- Resource constraints
- Configuration errors

**Troubleshooting Steps**:
1. Check available revisions:
   ```bash
   kubectl rollout history deployment/<deployment-name>
   ```
2. Describe the deployment:
   ```bash
   kubectl describe deployment <deployment-name>
   ```
3. Check events:
   ```bash
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

## Next Steps

In the next section, we'll explore DaemonSets, which ensure that all (or some) nodes run a copy of a pod.

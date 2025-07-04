# 7.5 Managing Update Strategy

## Introduction to Update Strategies

Kubernetes Deployments support different update strategies that control how updates are rolled out to your application. Understanding these strategies is crucial for managing application deployments effectively.

## Available Update Strategies

### 1. RollingUpdate (Default)

Gradually replaces old pods with new ones, ensuring continuous availability.

**Key Features:**
- Zero-downtime updates
- Configurable surge and unavailable settings
- Supports rollback

**Example Configuration:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

### 2. Recreate

Terminates all existing pods before creating new ones, causing a brief service interruption.

**Use Cases:**
- When your application can't run multiple versions simultaneously
- During major version upgrades
- When you need to ensure a clean state

**Example Configuration:**
```yaml
spec:
  strategy:
    type: Recreate
```

## Configuring Rolling Updates

### Understanding maxSurge and maxUnavailable

- **maxSurge**: The maximum number of pods that can be created over the desired number of pods during an update.
  - Can be a percentage (e.g., 25%) or an absolute number (e.g., 2)
  - Default is 25%

- **maxUnavailable**: The maximum number of pods that can be unavailable during the update.
  - Can be a percentage (e.g., 25%) or an absolute number (e.g., 1)
  - Default is 25%

### Practical Examples

#### Example 1: Conservative Update
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```
This ensures:
- Only one pod above the desired count is created at a time
- No pods are unavailable during the update
- Slow but safest update

#### Example 2: Aggressive Update
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 100%
    maxUnavailable: 50%
```
This allows:
- Creating up to 100% more pods than the desired count
- Up to 50% of pods can be unavailable during the update
- Faster update but with potential temporary service degradation

## Advanced Update Controls

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

### Setting Progress Deadlines

Kubernetes allows you to set a deadline for the rollout to make progress:

```yaml
spec:
  progressDeadlineSeconds: 600  # 10 minutes
  minReadySeconds: 5
```

If the deployment doesn't complete within this time, it will be marked as failed.

## Practical Example: Configuring Update Strategy

Let's create a deployment with a custom update strategy:

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
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
        image: nginx:1.19.10
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
```

## Monitoring and Troubleshooting

### Checking Rollout Status

```bash
# Watch the deployment status
kubectl rollout status deployment/nginx-deployment

# View deployment history
kubectl rollout history deployment/nginx-deployment

# View details of a specific revision
kubectl rollout history deployment/nginx-deployment --revision=2
```

### Common Issues and Solutions

#### Issue: Deployment Stuck

**Symptoms**:
- Rollout doesn't complete
- Some pods are in a bad state

**Troubleshooting Steps**:
1. Check deployment status:
   ```bash
   kubectl describe deployment nginx-deployment
   ```
2. Look at events:
   ```bash
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```
3. Check pod status:
   ```bash
   kubectl get pods -l app=nginx
   ```
4. Check pod logs:
   ```bash
   kubectl logs <pod-name>
   ```

#### Issue: Rollback Fails

**Possible Causes**:
- No previous revisions available
- Resource constraints
- Configuration errors

**Solution**:
1. Check available revisions:
   ```bash
   kubectl rollout history deployment/nginx-deployment
   ```
2. Manually edit the deployment if needed

## Best Practices

1. **Use Appropriate Strategy**: Choose between RollingUpdate and Recreate based on your application's needs
2. **Set Reasonable Timeouts**: Configure `progressDeadlineSeconds` to detect failed deployments
3. **Use Readiness Probes**: Ensure pods are ready before receiving traffic
4. **Test Rollbacks**: Regularly test rollback procedures
5. **Monitor Resource Usage**: Ensure you have enough resources for the update
6. **Use Pause/Resume**: For complex updates, consider pausing the deployment

## Next Steps

In the next section, we'll explore how to manage deployment history and perform rollbacks in more detail.

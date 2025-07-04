# 7.2 Managing Deployment Scalability

## Introduction to Scaling in Kubernetes

Scalability is a fundamental feature of Kubernetes that allows your applications to handle increased load by adjusting the number of running pod instances. This section covers both manual and automatic scaling techniques.

## Historical Context

- In early Kubernetes versions (pre-1.8), scaling was managed by ReplicationController
- With the introduction of the apps/v1 API, Deployments became the primary way to manage scaling
- Deployments internally use ReplicaSets for pod management

## Manual Scaling

### Using kubectl scale

The most straightforward way to scale a deployment:

```bash
# Scale a deployment to 5 replicas
kubectl scale deployment/my-web --replicas=5

# Scale using the deployment name
kubectl scale deployment my-web --replicas=3
```

### Creating Deployments with Specific Replicas

```bash
# Create a deployment with 3 replicas (Kubernetes 1.18+)
kubectl create deployment my-app --image=nginx --replicas=3
```

For versions before 1.18, you would need to create and then scale:

```bash
kubectl create deployment my-app --image=nginx
kubectl scale deployment my-app --replicas=3
```

### Editing Deployments

You can also edit the deployment directly:

```bash
kubectl edit deployment my-app
```

Then modify the `spec.replicas` field.

## Understanding ReplicaSets

Deployments manage ReplicaSets, which in turn manage Pods. You can view the ReplicaSets created by a deployment:

```bash
kubectl get replicasets -l app=my-app
```

## Practical Example: Scaling a Redis Deployment

1. First, create a Redis deployment:

```yaml
# redis-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.0
        ports:
        - containerPort: 6379
```

2. Apply the deployment:

```bash
kubectl apply -f redis-deploy.yaml
```

3. Scale the deployment:

```bash
kubectl scale deployment redis --replicas=3
```

4. Verify the scaling:

```bash
kubectl get pods -l app=redis
```

## Scaling Best Practices

1. **Resource Management**: Always set resource requests and limits
2. **Readiness Probes**: Ensure your pods properly report when they're ready to serve traffic
3. **Pod Disruption Budgets**: Use PDBs to ensure a minimum number of pods are always available during voluntary disruptions
4. **Horizontal Pod Autoscaler**: For automatic scaling based on metrics (covered in section 7.8)

## Common Issues and Solutions

### Issue: ReplicaSet Not Scaling

**Symptoms**:
- `kubectl get rs` shows 0/0 replicas
- No pods are being created

**Troubleshooting Steps**:
1. Check deployment status:
   ```bash
   kubectl describe deployment <deployment-name>
   ```
2. Look for events:
   ```bash
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```
3. Check for resource quotas:
   ```bash
   kubectl describe quota --namespace=<namespace>
   ```

### Issue: Pods Stuck in Pending State

**Possible Causes**:
- Insufficient cluster resources
- Node selectors/node affinity issues
- Resource quotas

**Troubleshooting**:
```bash
# Describe the pending pod
kubectl describe pod <pod-name>

# Check node resources
kubectl describe nodes
```

## Advanced Scaling Patterns

### Proportional Scaling

When updating a deployment, you can control how the scaling happens during the update:

```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

### Zero-Downtime Scaling

To ensure zero downtime when scaling down:

1. Use readiness probes
2. Configure proper termination grace periods
3. Use preStop hooks for graceful shutdown

## Monitoring Scaling

### Viewing Deployment Status

```bash
# Watch the deployment status
kubectl rollout status deployment/my-app

# View detailed deployment history
kubectl rollout history deployment/my-app
```

### Monitoring Resource Usage

```bash
# View resource usage for pods
kubectl top pods

# View resource usage for nodes
kubectl top nodes
```

## Next Steps

In the next section, we'll explore deployment updates in more detail, including rolling updates and rollbacks.

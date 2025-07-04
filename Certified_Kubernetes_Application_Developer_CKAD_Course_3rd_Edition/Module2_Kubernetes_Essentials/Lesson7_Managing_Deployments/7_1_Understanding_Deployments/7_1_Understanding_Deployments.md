# 7.1 Understanding Deployments

## What are Deployments?

Deployments are the recommended way to manage applications in Kubernetes. They provide a declarative way to manage Pods and ReplicaSets, offering features that enhance both scalability and reliability of your applications.

## Key Features

- **Scalability**: Easily scale the number of application instances
- **Updates**: Support for zero-downtime application updates
- **Self-healing**: Automatically restarts failed containers
- **Rollbacks**: Easy rollback to previous versions
- **Pause/Resume**: Pause deployments to apply multiple changes

## Creating a Deployment

### Basic Command
```bash
kubectl create deployment my-web --image=nginx --replicas=3
```

This command creates a deployment named `my-web` with 3 replicas of the nginx container.

> **Note**: The `--replicas` flag was added in Kubernetes 1.18. For earlier versions, you would need to scale the deployment manually after creation.

## Deployment Components

### Labels and Selectors

When you create a deployment, Kubernetes automatically adds labels to the deployment and its pods:

```yaml
metadata:
  labels:
    app: my-web
```

These labels are used by the deployment's selector to manage the pods:

```yaml
selector:
  matchLabels:
    app: my-web
```

### Pod Template

The deployment uses a pod template to create new pods:

```yaml
template:
  metadata:
    labels:
      app: my-web
  spec:
    containers:
    - name: nginx
      image: nginx
```

## Managing Deployments

### Viewing Deployments

```bash
# List all deployments
kubectl get deployments

# Get detailed information about a deployment
kubectl describe deployment my-web

# View all resources created by the deployment
kubectl get all -l app=my-web
```

### How Deployments Work

1. When you create a deployment, it creates a ReplicaSet
2. The ReplicaSet ensures the desired number of pods are running
3. If a pod fails or is deleted, the ReplicaSet creates a new one
4. The deployment manages updates by creating new ReplicaSets and scaling them appropriately

## Deployment vs Naked Pods

### Why Use Deployments?

- **Self-healing**: If a pod fails, the deployment replaces it
- **Scaling**: Easily scale the number of pods up or down
- **Updates**: Perform rolling updates with zero downtime
- **Rollbacks**: Revert to a previous version if something goes wrong

### Example: Naked Pod Limitation

```bash
# Create a naked pod
kubectl run my-pod --image=nginx

# Delete the pod
kubectl delete pod my-pod
```

When you delete a naked pod, it's gone permanently. With deployments, the pod would be recreated automatically.

## Common Operations

### Scaling a Deployment

```bash
# Scale to 5 replicas
kubectl scale deployment my-web --replicas=5

# Scale down to 2 replicas
kubectl scale deployment my-web --replicas=2
```

### Updating a Deployment

```bash
# Update the container image
kubectl set image deployment/my-web nginx=nginx:1.19
```

### Deleting a Deployment

```bash
# Delete a deployment
kubectl delete deployment my-web
```

## Best Practices

1. Always use deployments instead of naked pods
2. Use meaningful names for your deployments
3. Set resource requests and limits
4. Use proper labels and selectors
5. Consider using namespaces to organize your deployments

## Troubleshooting

### Common Issues

1. **Image Pull Errors**: Check if the image name and tag are correct
2. **CrashLoopBackOff**: Check container logs with `kubectl logs <pod-name>`
3. **Pending Pods**: Check resource availability with `kubectl describe pod <pod-name>`

### Useful Commands

```bash
# View deployment status
kubectl rollout status deployment/my-web

# View pod logs
kubectl logs -l app=my-web

# View events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Next Steps

In the next section, we'll explore how to manage deployment scalability in more detail.

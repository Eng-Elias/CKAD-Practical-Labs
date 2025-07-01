# 15.13 Using Quota

## Assignment
Create a namespace named `limited` with the following resource quotas:
- Maximum of 5 pods
- Total of 1000 millicores CPU
- Total of 2GB RAM

Then, create an nginx deployment in this namespace with:
- Name: `restrictginx`
- 3 replicas
- Each pod should request 64MB RAM with an upper limit of 256MB RAM

## Solution Walkthrough

### 1. Create the Namespace
```bash
kubectl create ns limited
```

### 2. Create Resource Quota
```bash
kubectl create quota limited-quota -n limited \
  --hard=pods=5,cpu=1,memory=2G
```

Verify the quota settings:
```bash
kubectl describe ns limited
```

### 3. Create the Deployment
```bash
kubectl create deploy restrictginx -n limited \
  --replicas=3 \
  --image=nginx
```

### 4. Set Resource Limits and Requests
First attempt (with only memory limits):
```bash
kubectl set resources -n limited deployment/restrictginx \
  --limits=memory=256Mi \
  --requests=memory=64Mi
```

### 5. Fix the CPU Quota Issue
Check the replica set status:
```bash
kubectl describe -n limited replicaset restrictginx-<hash>
```

You'll see an error about CPU quota. We need to set CPU requests and limits as well:

```bash
# Delete the existing deployment to start fresh
kubectl delete deploy restrictginx -n limited

# Recreate with CPU settings
kubectl create deploy restrictginx -n limited \
  --replicas=3 \
  --image=nginx

# Set both CPU and memory limits
kubectl set resources -n limited deployment/restrictnginx \
  --limits=cpu=200m,memory=256Mi \
  --requests=cpu=200m,memory=64Mi
```

### 6. Verify the Quota Usage
```bash
kubectl describe ns limited
```

## Common Issues and Solutions

1. **CPU Quota Not Set**
   - Error: `must specify cpu`
   - Solution: Ensure you set both CPU requests and limits when CPU quota is defined in the namespace

2. **Insufficient Quota**
   - If you see quota exceeded errors, check current usage:
     ```bash
     kubectl get resourcequota -n limited
     ```
   - Adjust your resource requests/limits or increase the namespace quota

3. **Pods Not Starting**
   - Check pod events:
     ```bash
     kubectl get events -n limited
     ```
   - Verify resource availability on nodes:
     ```bash
     kubectl describe nodes
     ```

## Exam Tips

1. **Resource Units**
   - CPU: 1000m = 1 CPU core
   - Memory: 1G = 1000MB, 1Gi = 1024MiB
   - Always specify units (e.g., 256Mi, 500m)

2. **Quota Best Practices**
   - Set both requests and limits
   - Start with reasonable defaults
   - Monitor usage before setting strict quotas

3. **Troubleshooting**
   - Use `kubectl describe` to see quota limits and usage
   - Check pod events for quota-related errors
   - Remember that some resources (like CPU) require both limits and requests to be set

## Cleanup
```bash
kubectl delete ns limited
```

## Key Takeaways
- Resource quotas help manage resource usage in namespaces
- Always set both requests and limits for resources with quotas
- Monitor quota usage to prevent deployment failures
- Remember to include CPU specifications when CPU quotas are in place

# 15.14 Creating Canary Deployments

## Assignment
Create a canary deployment with the following requirements:
1. Initial deployment:
   - Name: `my-web`
   - Image: `nginx:1.14`
   - Replicas: 3
   - Accessible via a service named `canary` (NodePort type)
2. Update strategy:
   - Deploy a new version using `nginx:latest`
   - Traffic split: 40% new version, 60% old version

## Solution Walkthrough

### 1. Create the Initial Deployment
```bash
kubectl create deploy my-web --image=nginx:1.14 --replicas=3
```

### 2. Add Canary Labels
We need to add a `type: canary` label to both the deployment and pods:

```bash
# Add label to the deployment
kubectl label deploy my-web type=canary

# Verify the deployment and pods have the label
kubectl get all --selector type=canary
```

### 3. Create the Canary Service
```bash
kubectl expose deploy my-web --name=canary --port=80 --selector type=canary

# Verify the service
kubectl describe svc canary
```

### 4. Deploy the Canary Version
Create a new deployment with the updated image and adjusted replica count:

```bash
# Create new deployment with updated image
kubectl create deploy my-web-v2 --image=nginx:latest --replicas=2

# Add the same canary label to the new deployment
kubectl label deploy my-web-v2 type=canary
```

### 5. Verify the Traffic Split
```bash
# Check all pods
kubectl get pods -o wide

# Check service endpoints
kubectl get endpoints canary

# The output should show 5 endpoints in total:
# - 3 from the original deployment (nginx:1.14)
# - 2 from the new deployment (nginx:latest)
# This gives us approximately 40% traffic to the new version
```

## Common Issues and Solutions

1. **Incorrect Label Selector**
   - Ensure both deployments and their pods have the same `type: canary` label
   - Check with: `kubectl get all --selector type=canary`

2. **Service Not Routing Traffic**
   - Verify the service selector matches the pod labels:
     ```bash
     kubectl describe svc canary | grep -i selector
     ```
   - Check that pods are in the same namespace as the service

3. **Uneven Traffic Distribution**
   - The ratio is controlled by the number of replicas in each deployment
   - Adjust replica counts to achieve desired traffic split
   - For more precise control, consider using an Ingress controller with traffic splitting

## Exam Tips

1. **Label Management**
   - Use consistent labels across related resources
   - Remember to label both deployments and pods for canary deployments

2. **Verification**
   - Always verify:
     - Pod counts in each deployment
     - Service endpoints
     - Labels on all resources

3. **Cleanup**
   ```bash
   # Delete deployments
   kubectl delete deploy my-web my-web-v2
   
   # Delete service
   kubectl delete svc canary
   ```

## Key Takeaways
- Canary deployments allow gradual rollouts of new versions
- Traffic is split between old and new versions based on replica counts
- Labels are crucial for service discovery and traffic routing
- Always verify the traffic split before considering the rollout complete
- Monitor the new version's performance before completing the rollout
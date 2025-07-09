# CKAD Practice Question 21: Deployments with Resource Limits and Service Accounts

## Question
Team Neptune needs three pods with the following specifications:

**Requirements**:
- **Deployment Name**: `neptune-10ab`
- **Namespace**: `neptune`
- **Container Image**: `httpd:2.4-alpine`
- **Container Name**: `neptune-pod-10ab`
- **Resource Requirements**:
  - Memory Request: 20MB
  - Memory Limit: 50MB
- **Service Account**: `neptune-sa-v2`

## Solution

### Step 1: Create the Deployment YAML
Create a file named `deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: neptune-10ab
  namespace: neptune
  labels:
    app: neptune
spec:
  replicas: 3
  selector:
    matchLabels:
      app: neptune
  template:
    metadata:
      labels:
        app: neptune
    spec:
      serviceAccountName: neptune-sa-v2  # Using the specified service account
      containers:
      - name: neptune-pod-10ab  # Container name as specified
        image: httpd:2.4-alpine  # Using the specified image
        resources:
          requests:
            memory: "20Mi"  # 20MB memory request
          limits:
            memory: "50Mi"  # 50MB memory limit
```

### Step 2: Apply the Deployment
```bash
# Create the namespace if it doesn't exist
kubectl create namespace neptune --dry-run=client -o yaml | kubectl apply -f -

# Apply the deployment
kubectl apply -f deployment.yaml

# Verify the deployment
kubectl -n neptune get deployment neptune-10ab

# Check the pods
kubectl -n neptune get pods -l app=neptune
```

### Step 3: Verify the Configuration
```bash
# Check pod resource limits
kubectl -n neptune describe pod <pod-name> | grep -A 5 "Limits"

# Verify the service account
kubectl -n neptune get pod <pod-name> -o jsonpath='{.spec.serviceAccountName}'

# Check container name
kubectl -n neptune get pod <pod-name> -o jsonpath='{.spec.containers[0].name}'
```

## Explanation

### Key Concepts
1. **Deployments**:
   - Manage a replicated application
   - Ensure the desired number of pods are running
   - Allow for rolling updates and rollbacks

2. **Resource Limits**:
   - `requests`: Guaranteed minimum resources
   - `limits`: Maximum resources a container can use
   - Memory is specified in bytes, with common suffixes (Ki, Mi, Gi)

3. **Service Accounts**:
   - Provides an identity for processes in a pod
   - Controls API access permissions
   - Must exist before being referenced in a pod spec

### Why This Solution Works
- The deployment ensures three replicas of the pod are always running
- Resource limits prevent containers from using too much memory
- The specified service account is used for API authentication
- The container is named as required

### Exam Tips
1. **Resource Units**:
   - 1 MiB = 1024 KiB
   - 1 MB = 1000 KB
   - Always specify units (e.g., Mi, Gi) for clarity

2. **Service Accounts**:
   - Must exist in the same namespace
   - Default service account is used if none specified
   - Can be created with `kubectl create serviceaccount`

3. **Container Naming**:
   - Must be unique within a pod
   - Follow DNS-1123 naming conventions
   - No uppercase letters allowed

### Common Mistakes to Avoid
- Incorrect memory units (MB vs MiB)
- Forgetting to specify the namespace
- Incorrect indentation in YAML
- Using uppercase letters in container names
- Not verifying the service account exists

## Additional Practice
1. Create a deployment with CPU limits
2. Configure liveness and readiness probes
3. Set up pod anti-affinity rules
4. Configure environment variables
5. Add volume mounts to the deployment

## Related Commands
```bash
# Create a service account
kubectl -n <namespace> create serviceaccount <name>

# Get detailed pod information
kubectl -n <namespace> describe pod <pod-name>

# Check deployment status
kubectl -n <namespace> rollout status deployment/<deployment-name>

# Scale a deployment
kubectl -n <namespace> scale deployment <name> --replicas=5

# View pod logs
kubectl -n <namespace> logs <pod-name> -c <container-name>

# Edit a deployment
kubectl -n <namespace> edit deployment <name>
```

# CKAD Practice Question 7: Migrating a Pod Between Namespaces

## Question
Team Neptune is taking over control of an e-commerce web server from Team Saturn. The only information available is that the e-commerce system is called "My Happy Shop".

**Tasks**:
1. Locate the pod running the "My Happy Shop" application in the `saturn` namespace
2. Migrate the pod to the `neptune` namespace
3. Ensure the application continues to function in the new namespace

**Important Notes**:
- You can shut down and restart the pod if needed
- The application is not currently serving customers, so downtime is acceptable
## Solution

### Step 1: Locate the Target Pod
```bash
# List all resources in the saturn namespace
kubectl get all -n saturn

# Search for the pod containing "My Happy Shop" in its configuration
kubectl get pods -n saturn -o yaml | grep -A 3 -B 3 "My Happy Shop"

# Once identified, get the full pod definition
kubectl get pod <pod-name> -n saturn -o yaml > my-happy-shop.yaml
```

### Step 2: Prepare the Pod Definition for Migration
Edit the `my-happy-shop.yaml` file to:
1. Change the namespace to `neptune`
2. Remove unnecessary fields that might cause issues:
   - `status` section
   - `metadata.uid`
   - `metadata.resourceVersion`
   - `metadata.selfLink`
   - `metadata.creationTimestamp`
   - `metadata.annotations` (especially `kubectl.kubernetes.io/last-applied-configuration`)
   - `spec.nodeName` (to allow scheduling on any available node)
   - `status`-related fields in the container status

### Step 3: Create the Pod in the New Namespace
```bash
# First, ensure the target namespace exists
kubectl create namespace neptune 2>/dev/null || true

# Create the pod in the new namespace
kubectl apply -f my-happy-shop.yaml -n neptune

# Verify the pod is running in the new namespace
kubectl get pods -n neptune
```

### Step 4: Clean Up (Optional)
```bash
# Delete the original pod from the saturn namespace
kubectl delete pod <pod-name> -n saturn
```

## Explanation

### Key Concepts
1. **Namespaces**: Logical partitions in Kubernetes that allow resources to be organized and managed separately
2. **Pod Migration**: The process of moving a pod from one namespace to another
3. **Resource Definitions**: Kubernetes resources are defined in YAML/JSON format and can be exported/imported
4. **Immutable Fields**: Some fields in Kubernetes resources cannot be modified after creation

### Why This Solution Works
- Exporting the pod definition allows us to modify it before recreation
- Removing status and metadata fields ensures a clean creation in the new namespace
- Creating the pod in the new namespace before deleting the old one ensures minimal downtime
- The solution maintains all the original pod's configuration and specifications

### Exam Tips
1. **Namespace Management**: Always verify the current namespace and specify it with `-n`
2. **Resource Inspection**: Use `kubectl get <resource> -o yaml` to get complete resource definitions
3. **Clean Resource Definitions**: Remove status and metadata fields when moving resources
4. **Verification**: Always verify the pod is running correctly in the new namespace
5. **Documentation**: Know how to use `kubectl explain` to understand resource fields

### Common Mistakes to Avoid
- Forgetting to change the namespace in the YAML file
- Not removing immutable fields before recreation
- Deleting the original pod before verifying the new one is running
- Missing important configuration details during the migration
- Not verifying the application functionality after migration

## Additional Practice
1. Migrate a deployment with multiple replicas between namespaces
2. Move resources along with their associated services and configmaps
3. Create a script to automate the migration of multiple resources
4. Handle storage migration (PV/PVC) between namespaces
5. Set up network policies in the new namespace

## Related Commands
```bash
# Get detailed information about a pod
kubectl describe pod <pod-name> -n <namespace>

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Execute a command in the pod
kubectl exec -it <pod-name> -n <namespace> -- <command>

# Check events in a namespace
kubectl get events -n <namespace>

# Verify all resources in a namespace
kubectl get all -n <namespace>
```

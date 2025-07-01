# 15.16 Managing ServiceAccount

## Assignment
Create a pod with the following specifications:
- Name: `allaccess`
- Image: `busybox`
- Command: `sleep 3600`
- Service Account: `allaccess` (needs to be created)

## Solution Walkthrough

### 1. Create the ServiceAccount
```bash
kubectl create serviceaccount allaccess
```

### 2. Generate the Pod Manifest
Generate a basic pod manifest using `kubectl run` with the `--dry-run=client` flag:

```bash
kubectl run allaccess --image=busybox --dry-run=client -o yaml -- sleep 3600 > pod-with-sa.yaml
```

### 3. Edit the Pod Manifest
Edit the generated `pod-with-sa.yaml` file to include the service account:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: allaccess
spec:
  serviceAccountName: allaccess  # This assigns the service account to the pod
  containers:
  - name: allaccess
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      allowPrivilegeEscalation: false
```

### 4. Apply the Configuration
```bash
kubectl apply -f pod-with-sa.yaml
```

### 5. Verify the Configuration
Check that the pod is running with the correct service account:

```bash
# Check pod status
kubectl get pod allaccess

# Verify the service account
kubectl get pod allaccess -o yaml | grep -i serviceAccount
```

## Common Issues and Solutions

1. **ServiceAccount Not Found**
   - Error: `serviceaccount "allaccess" not found`
   - Solution: Ensure you've created the service account before creating the pod

2. **Incorrect YAML Format**
   - Make sure `serviceAccountName` is at the pod spec level (not under containers)
   - Check for proper indentation in the YAML file

3. **Permission Issues**
   - If the pod fails to start, check the service account's RBAC permissions
   - Verify with: `kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:allaccess`

## Exam Tips

1. **ServiceAccount Basics**
   - Service accounts authenticate processes running in pods
   - Default service account is used if none is specified
   - Service accounts are namespace-scoped

2. **Verification**
   Always verify your configuration by:
   ```bash
   # Check service accounts in the current namespace
   kubectl get serviceaccounts
   
   # Get pod details including service account
   kubectl describe pod <pod-name>
   ```

3. **Cleanup**
   ```bash
   # Delete the pod
   kubectl delete pod allaccess
   
   # Delete the service account (optional)
   kubectl delete serviceaccount allaccess
   ```

## Key Takeaways
- Service accounts provide an identity for processes running in pods
- Always create the service account before creating pods that use it
- The `serviceAccountName` field in the pod spec assigns a service account
- Verify service account assignment using `kubectl describe pod`
- Remember that service accounts are different from user accounts (they're for processes, not people)
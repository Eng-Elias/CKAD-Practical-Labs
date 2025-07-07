# CKAD Practice Question 2: Create a Pod and Check Status

## Question
**Task 1:** Create a single-container Pod in the default namespace. The pod should be named `pod1` and the container should be named `pod1-container`. The container should use the `httpd:2.4.41-alpine` image.

**Task 2:** Create a command that outputs the status of this specific pod and save it to the file `/opt/course/2/status-check.sh`. The command should use `kubectl` (not the `k` alias).

## Solution

### Task 1: Create the Pod

#### Method 1: Using `kubectl run` with `--dry-run`
```bash
# Generate the Pod manifest
kubectl run pod1 --image=httpd:2.4.41-alpine --dry-run=client -o yaml > 2.yaml

# Edit the generated YAML to update the container name
vim 2.yaml
# change container name from pod1 to pod1-container

kubectl create -f 2.yaml

# Create the pod
kubectl apply -f 2.yaml

# Verify the pod is running
kubectl get pods
# or 
kubectl get po

# get pod details
kubectl describe pod pod1

# get pod status
kubectl describe pod pod1 | grep -i "status:"

# get pod status using jsonpath
kubectl -n default get pod pod1 -o jsonpath="{.status.phase}"
```

#### Method 2: Directly create the Pod with a YAML file
```yaml
# pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: default
spec:
  containers:
  - name: pod1-container
    image: httpd:2.4.41-alpine
  restartPolicy: Always
```

```bash
# Apply the YAML file
kubectl apply -f pod1.yaml
```

### Task 2: Create the Status Check Script

#### Method 1: Using JSON Path
```bash
# Create the directory if it doesn't exist
mkdir -p /opt/course/2/

# Create the status check script
echo 'kubectl get pod pod1 -n default -o jsonpath="{.status.phase}"' > /opt/course/2/status-check.sh

# Make the script executable
chmod +x /opt/course/2/status-check.sh

# Verify the script works
/opt/course/2/status-check.sh
```

#### Method 2: Using `kubectl describe` and `grep`
```bash
echo 'kubectl describe pod pod1 -n default | grep -i "status:" | head -1' > /opt/course/2/status-check.sh
chmod +x /opt/course/2/status-check.sh
```

## Explanation

### Key Concepts
1. **Pod Creation**: A Pod is the smallest deployable unit in Kubernetes that can contain one or more containers.
2. **Container Naming**: Each container in a Pod must have a unique name.
3. **JSONPath**: A query language for JSON that allows extracting specific fields from the output.
4. **Scripting**: Creating executable scripts to automate repetitive tasks.

### Why These Solutions Work
- The first method uses `kubectl run` with `--dry-run` to generate a template, which is then modified and applied.
- The second method directly creates a YAML manifest with all required specifications.
- The status check script uses `kubectl` with JSONPath to extract just the pod status.

### Exam Tips
1. **Use `--dry-run`**: Generate YAML/JSON manifests quickly using `--dry-run=client -o yaml`.
2. **JSONPath**: Learn basic JSONPath queries for extracting specific fields from `kubectl` output.
3. **Script Permissions**: Don't forget to make scripts executable with `chmod +x`.
4. **Explicit Namespace**: Always specify the namespace with `-n` to avoid context switching issues.
5. **Verification**: Always verify your resources after creation with `kubectl get` or `kubectl describe`.

### Common Mistakes to Avoid
- Forgetting to make the script executable
- Using the `k` alias when the question specifies to use `kubectl`
- Not verifying the pod is running before creating the status check
- Incorrect JSONPath syntax when extracting fields

## Additional Practice
1. Create a pod with multiple containers
2. Extract different pod status fields using JSONPath
3. Create a script that checks if a pod is ready (not just running)
4. Add error handling to the status check script

## Related Commands
```bash
# Get pod details in YAML format
kubectl get pod pod1 -o yaml

# Watch pod status
kubectl get pod pod1 -w

# Get pod logs
kubectl logs pod1 -c pod1-container

# Delete the pod
kubectl delete pod pod1
```

# CKAD Practice Question 6: Configuring Readiness Probes

## Question
Create a single pod named `pod6` in the `default` namespace with the following specifications:

1. **Image**: `busybox:1.31.0`
2. **Readiness Probe**:
   - Execute command: `cat /tmp/ready`
   - Initial delay: 5 seconds
   - Period: 10 seconds
3. **Container Command**:
   - First, create the file: `touch /tmp/ready`
   - Then sleep for a day: `sleep 86400`

## Solution

### Method 1: Using kubectl run with --dry-run

1. **Generate the base pod manifest**:
```bash
kubectl run pod6 \
  --image=busybox:1.31.0 \
  --dry-run=client \
  -o yaml > pod6.yaml
```

2. **Edit the generated YAML** to add the readiness probe and command:
```yaml
# pod6.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox:1.31.0
    command: ["sh", "-c"]
    args: ["touch /tmp/ready && sleep 86400"]
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5
      periodSeconds: 10
```

3. **Create the pod**:
```bash
kubectl apply -f pod6.yaml
```

4. **Verify the pod status**:
```bash
kubectl get pod pod6
kubectl describe pod pod6
```

### Method 2: Directly create the pod
```bash
kubectl run pod6 \
  --image=busybox:1.31.0 \
  --restart=Never \
  --command -- sh -c "touch /tmp/ready && sleep 86400" \
  --dry-run=client \
  -o yaml | \
  kubectl apply -f -

# Add the readiness probe using kubectl patch
kubectl patch pod pod6 -p '
{
  "spec": {
    "containers": [
      {
        "name": "pod6",
        "readinessProbe": {
          "exec": {
            "command": ["cat", "/tmp/ready"]
          },
          "initialDelaySeconds": 5,
          "periodSeconds": 10
        }
      }
    ]
  }
}'
```

## Explanation

### Key Concepts
1. **Readiness Probes**: Determine if a container is ready to serve traffic
2. **Initial Delay**: Time to wait before performing the first probe
3. **Period**: How often to perform the probe after the initial delay
4. **Exec Action**: Executes a command inside the container

### Why This Solution Works
- The container creates `/tmp/ready` immediately on startup
- The readiness probe checks for this file's existence
- The initial delay gives the container time to create the file
- The period ensures the probe runs regularly

### Exam Tips
1. **Probe Types**: Know the difference between readiness, liveness, and startup probes
2. **Timing**: Understand the relationship between `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds`, `successThreshold`, and `failureThreshold`
3. **Debugging**: Use `kubectl describe` to see probe status and container state
4. **YAML Structure**: Ensure proper indentation for the probe configuration
5. **Command Format**: Remember to use array format for commands and arguments

### Common Mistakes to Avoid
- Forgetting to set the initial delay (container might fail before creating the file)
- Using incorrect file paths in the probe
- Not understanding the difference between readiness and liveness probes
- Forgetting to set the container command to create the required file
- Incorrect YAML indentation

## Additional Practice
1. Create a pod with both readiness and liveness probes
2. Configure a TCP socket probe instead of an exec probe
3. Set up a pod that fails its readiness probe after a certain condition
4. Create a pod with a startup probe that overrides readiness/liveness probes initially
5. Debug a pod that's not becoming ready

## Related Commands
```bash
# Check pod events for probe failures
kubectl describe pod <pod-name>

# View container logs
kubectl logs <pod-name>

# Execute a command in the container
kubectl exec -it <pod-name> -- sh

# Get pod details in YAML format
kubectl get pod <pod-name> -o yaml

# Delete the pod
kubectl delete pod pod6
```

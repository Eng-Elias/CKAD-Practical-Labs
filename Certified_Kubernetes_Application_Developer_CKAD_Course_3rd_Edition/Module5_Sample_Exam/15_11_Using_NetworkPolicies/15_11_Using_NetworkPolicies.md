# 15.11 Using NetworkPolicies

This section provides a solution for the sample exam assignment on controlling pod-to-pod communication using NetworkPolicies.

## Task

1. Create a YAML file named `my-nw-policy.yaml` that includes:
   - An nginx pod with default settings
   - A busybox pod that runs `sleep 3600`
   - A NetworkPolicy that allows traffic from the busybox pod to the nginx pod
2. The nginx pod should only accept traffic from pods with the label `role: frontend`
3. The busybox pod should be able to access the nginx pod after proper labeling

---

## Solution Walkthrough

This task involves creating pods and configuring NetworkPolicies to control pod communication.

### Exam Tip: What You Don't Need to Do

*   **Install Network Plugins:** The cluster should already have a network plugin that supports NetworkPolicy (like Calico or Weave Net)
*   **Edit System Files:** No need to modify any system configurations or host files

### 1. Create the Pods and NetworkPolicy

First, let's create the YAML manifest with all required resources:

```yaml
# my-nw-policy.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nwpnginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: nwpbusybox
  labels:
    access: allowed
spec:
  containers:
  - name: busybox
    image: busybox
    args: 
    - sleep
    - "3600"
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

### 2. Apply the Configuration

```bash
# Apply the configuration
kubectl apply -f my-nw-policy.yaml

# Verify the pods are running
kubectl get pods

# Expose the nginx pod to test connectivity
kubectl expose pod nwpnginx --port=80
```

### 3. Test the NetworkPolicy

```bash
# First test without the required label (should fail)
kubectl exec -it nwpbusybox -- wget --spider --timeout=1 nwpnginx

# Add the required label to the busybox pod
kubectl label pod nwpbusybox role=frontend

# Test again (should succeed)
kubectl exec -it nwpbusybox -- wget --spider --timeout=1 nwpnginx
```

### Common Issues and Solutions

1. **Boolean Values in Labels**
   - Incorrect: `access: true`
   - Correct: `access: allowed`
   
2. **NetworkPolicy Not Working**
   - Ensure your Kubernetes cluster has a network plugin that supports NetworkPolicy (e.g., Calico, Weave Net)
   - Verify pod labels match exactly what's specified in the NetworkPolicy

3. **Testing Connectivity**
   - Use `wget --spider` for simple connectivity tests
   - Check logs if the nginx pod isn't responding

### Verification

To verify everything is working correctly:

1. The busybox pod should be able to connect to the nginx pod after adding the `role: frontend` label
2. Other pods without the label should not be able to connect
3. Check NetworkPolicy status:
   ```bash
   kubectl describe networkpolicy test-network-policy
   ```

## Cleanup

```bash
kubectl delete -f my-nw-policy.yaml
kubectl delete svc nwpnginx
```
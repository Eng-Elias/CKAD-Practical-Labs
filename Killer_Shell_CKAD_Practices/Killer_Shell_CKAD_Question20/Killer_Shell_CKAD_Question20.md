# CKAD Practice Question 20: Network Policies for Egress Control

## Question
In the `venus` namespace, there are two deployments named `api` and `frontend`. Both deployments are exposed inside the cluster using services.

**Tasks**:
1. Create a NetworkPolicy named `np1` that:
   - Applies to pods in the `frontend` deployment
   - Restricts outgoing TCP connections to only the `api` deployment
   - Allows DNS resolution (UDP and TCP on port 53)
2. Test the network policy by:
   - Verifying access to the API service
   - Testing DNS resolution
   - Confirming external access is blocked

**Requirements**:
- Use appropriate pod selectors
- Ensure DNS resolution continues to work
- Test connectivity using `wget` from a frontend pod

## Solution

### Step 1: Verify Current Connectivity
```bash
# List all resources in the venus namespace
kubectl -n venus get all

# Test connectivity from frontend to api
kubectl -n venus run test-connectivity --image=busybox --rm -it --restart=Never -- \
  wget -qO- http://api:2222

# Test external connectivity (should work before applying the policy)
kubectl -n venus run test-external --image=busybox --rm -it --restart=Never -- \
  wget -qO- http://www.google.com
```

### Step 2: Create the Network Policy
Create a file named `network-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np1
  namespace: venus
spec:
  podSelector:
    matchLabels:
      id: frontend  # Applies to frontend pods
  policyTypes:
  - Egress
  egress:
  # Allow DNS resolution (TCP and UDP on port 53)
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  # Allow egress to api pods on port 2222
  - to:
    - podSelector:
        matchLabels:
          id: api  # Target api pods
    ports:
    - port: 2222
      protocol: TCP
```

### Step 3: Apply the Network Policy
```bash
# Apply the network policy
kubectl apply -f network-policy.yaml

# Verify the network policy was created
kubectl -n venus get networkpolicies
kubectl -n venus describe networkpolicy np1
```

### Step 4: Test the Network Policy
```bash
# Test connectivity to api (should work)
kubectl -n venus exec -it $(kubectl -n venus get pod -l id=frontend -o name | head -n 1) -- \
  wget -qO- --timeout=3 http://api:2222 || echo "Connection failed"

# Test DNS resolution (should work)
kubectl -n venus exec -it $(kubectl -n venus get pod -l id=frontend -o name | head -n 1) -- \
  nslookup kubernetes.default.svc.cluster.local

# Test external connectivity (should be blocked)
kubectl -n venus exec -it $(kubectl -n venus get pod -l id=frontend -o name | head -n 1) -- \
  wget -qO- --timeout=3 http://www.google.com || echo "Connection blocked (expected)"
```

## Explanation

### Key Concepts
1. **Network Policies**:
   - Control traffic flow between pods and network endpoints
   - Work with the Container Network Interface (CNI) plugin
   - Are namespace-scoped

2. **Pod Selectors**:
   - `podSelector`: Selects pods to which the policy applies
   - `namespaceSelector`: Can be used to select pods in other namespaces
   - `ipBlock`: For allowing/denying specific IP ranges

3. **Policy Types**:
   - `Ingress`: Controls incoming traffic to pods
   - `Egress`: Controls outgoing traffic from pods

### Why This Solution Works
- The policy targets pods with label `id: frontend`
- It allows DNS resolution (UDP/TCP on port 53) which is essential for service discovery
- It specifically allows TCP traffic to pods with label `id: api` on port 2222
- All other egress traffic is denied by default

### Exam Tips
1. **Default Deny**: Network policies are deny-all by default
2. **DNS Requirements**: Always allow DNS (port 53) when restricting egress
3. **Testing**: Verify both allowed and denied paths
4. **Order Matters**: Rules are evaluated in order, first match wins
5. **Namespace Awareness**: Network policies are namespaced resources

### Common Mistakes to Avoid
- Forgetting to allow DNS traffic
- Incorrect pod selectors
- Not testing both allowed and denied paths
- Forgetting to specify the namespace
- Not verifying the CNI plugin supports NetworkPolicy

## Additional Practice
1. Create a network policy that allows ingress only from specific namespaces
2. Implement a default-deny-all policy for a namespace
3. Create a policy that allows specific IP ranges
4. Test network policies with different CNI plugins
5. Implement multi-tenant network isolation

## Related Commands
```bash
# Get network policies in a namespace
kubectl -n <namespace> get networkpolicies

# Describe a specific network policy
kubectl -n <namespace> describe networkpolicy <name>

# Check if a pod is affected by any network policies
kubectl -n <namespace> get pod <pod-name> -o yaml | grep -A 10 networkPolicy

# Check if the CNI plugin supports NetworkPolicy
kubectl get pods -n kube-system | grep -i cni

# Check network policy events
kubectl -n <namespace> get events --field-selector involvedObject.kind=NetworkPolicy
```

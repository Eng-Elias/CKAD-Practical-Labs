# 8.6 Understanding and Configuring NetworkPolicy

## Introduction to Network Policies

### What are Network Policies?
- Kubernetes resources that control traffic flow between pods
- Act as a firewall for your cluster
- Implemented by the network plugin (e.g., Calico, Cilium, Weave Net)

### Default Behavior
- If no policies exist, all traffic is allowed
- When policies are defined, they are additive (no "deny" rules)
- Traffic is allowed if any policy allows it

## Core Concepts

### Selectors
- **podSelector**: Selects pods to apply the policy to
- **namespaceSelector**: Selects namespaces to apply the policy to
- **ipBlock**: Specifies IP address ranges

### Policy Types
- **Ingress**: Controls incoming traffic to pods
- **Egress**: Controls outgoing traffic from pods

## Basic Network Policy Examples

### 1. Deny All Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 2. Allow All Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
  egress:
  - {}
```

## Advanced Network Policy Examples

### 3. Allow Traffic Between Specific Pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

### 4. Allow Traffic from Specific Namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
```

### 5. Allow Egress to External Services
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-external
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: external-client
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 192.168.0.0/16
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
```

## Practical Implementation

### 1. Verify Network Plugin Support
```bash
kubectl get pods -n kube-system | grep -E 'flannel|calico|weave|cilium'
```

### 2. Apply Network Policies
```bash
# Apply a network policy
kubectl apply -f network-policy.yaml

# List network policies
kubectl get networkpolicies --all-namespaces

# Describe a specific policy
kubectl describe networkpolicy <policy-name> -n <namespace>
```

### 3. Test Network Policies
```bash
# Run a test pod
kubectl run test-pod --image=nginx --labels=app=test

# Test connectivity
kubectl exec -it test-pod -- curl <target-service>
```

## Best Practices

### 1. Start with Default Deny
- Begin with a default-deny policy
- Gradually add allow rules
- Test after each change

### 2. Label Your Resources
- Use consistent labels
- Label namespaces appropriately
- Document your labeling scheme

### 3. Use Specific Selectors
- Be as specific as possible
- Avoid using empty selectors
- Use namespace selectors for cross-namespace policies

### 4. Monitor and Audit
- Log network traffic
- Monitor policy violations
- Regularly review and update policies

## Troubleshooting

### Common Issues
1. **Policies Not Working**
   - Verify network plugin supports NetworkPolicy
   - Check for typos in selectors
   - Ensure pods have the correct labels

2. **Unexpected Traffic Blocking**
   - Check for conflicting policies
   - Verify namespace selectors
   - Check for default deny policies

3. **DNS Resolution Issues**
   - Ensure DNS is allowed in egress rules
   - Check kube-dns service endpoints
   - Verify network plugin DNS integration

### Diagnostic Commands
```bash
# Check if network plugin is running
kubectl get pods -n kube-system

# Check network policies
kubectl get networkpolicies --all-namespaces

# Check pod labels
kubectl get pods --show-labels

# Check network policy logs (plugin specific)
kubectl logs -n kube-system <network-plugin-pod>
```

## Advanced Scenarios

### Multi-Tier Applications
```yaml
# web -> app -> db architecture
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-to-app
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      tier: app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: web
    ports:
    - protocol: TCP
      port: 8080
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-to-db
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      tier: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: app
    ports:
    - protocol: TCP
      port: 5432
```

### Egress to External Services
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-https
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: external-api-client
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: TCP
      port: 443
```

## Next Steps
In the next section, we'll explore Ingress controllers and how to manage external access to your services.

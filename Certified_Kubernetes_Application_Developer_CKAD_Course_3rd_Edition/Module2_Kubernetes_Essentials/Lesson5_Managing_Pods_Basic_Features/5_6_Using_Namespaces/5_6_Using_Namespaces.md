# 5.6 Using Namespaces

## What are Namespaces?
Namespaces in Kubernetes provide a mechanism for isolating groups of resources within a single cluster. They are virtual clusters backed by the same physical cluster.

### Key Characteristics
- **Logical Separation**: Divide cluster resources between multiple users/teams
- **Resource Quotas**: Set resource limits per namespace
- **Access Control**: Apply RBAC policies per namespace
- **Scoped Naming**: Resource names need to be unique within a namespace

## Default Namespaces

Kubernetes comes with several built-in namespaces:

1. **default**: The default namespace for objects with no other namespace
2. **kube-system**: For system components (pods, services, etc.)
3. **kube-public**: Contains resources that should be readable by all users
4. **kube-node-lease**: Holds node lease objects for node heartbeats

## Working with Namespaces

### Listing Namespaces
```bash
# List all namespaces
kubectl get namespaces
# or
kubectl get ns

# Get details about a specific namespace
kubectl describe namespace <namespace-name>
```

### Creating a Namespace

#### Using CLI
```bash
kubectl create namespace <namespace-name>
```

#### Using YAML
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
```
Apply with:
```bash
kubectl apply -f namespace.yaml
```

### Setting the Default Namespace
```bash
# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace-name>

# Verify current namespace
kubectl config view --minify --output 'jsonpath={..namespace}'
```

### Working with Resources in Namespaces

#### Creating Resources in a Namespace
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace  # Specify namespace here
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
```

#### Or specify namespace at the command line
```bash
kubectl apply -f pod.yaml -n my-namespace
```

#### Viewing Resources in a Namespace
```bash
# List pods in a specific namespace
kubectl get pods -n <namespace-name>

# List all resources in a namespace
kubectl get all -n <namespace-name>

# List resources across all namespaces
kubectl get pods --all-namespaces
```

## Advanced Namespace Features

### Resource Quotas
```yaml
# quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "10"
```

### Limit Ranges
```yaml
# limits.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: my-namespace
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

### Network Policies
```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: my-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## Managing Access with RBAC

### Role and RoleBinding (Namespace-scoped)
```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Namespace Best Practices

1. **Use Meaningful Names**: Choose names that reflect the purpose (e.g., dev, staging, production)
2. **Implement Resource Quotas**: Prevent resource exhaustion
3. **Use Network Policies**: Control traffic between namespaces
4. **Apply RBAC**: Restrict access based on roles
5. **Monitor Usage**: Keep an eye on resource consumption
6. **Clean Up**: Delete unused namespaces to free resources

## Common Operations

### Deleting a Namespace
```bash
# Delete a namespace and all its resources
kubectl delete namespace <namespace-name>

# Force delete if stuck in 'Terminating' state
kubectl get namespace <namespace-name> -o json > ns.json
# Edit ns.json to remove "kubernetes" from finalizers array
kubectl replace --raw "/api/v1/namespaces/<namespace-name>/finalize" -f ns.json
```

### Exporting Resources from a Namespace
```bash
# Export all resources from a namespace
export NAMESPACE=my-namespace
kubectl get all -n $NAMESPACE -o yaml > $NAMESPACE-backup.yaml

# Export specific resource types
kubectl get deploy,svc,pvc -n $NAMESPACE -o yaml > $NAMESPACE-resources.yaml
```

### Moving Resources Between Namespaces
```bash
# Export resource
kubectl get <resource-type> <resource-name> -n <source-namespace> -o yaml > resource.yaml

# Edit the namespace in resource.yaml
sed -i 's/namespace: source-namespace/namespace: target-namespace/g' resource.yaml

# Apply to new namespace
kubectl apply -f resource.yaml

# Delete from old namespace (after verifying the new one works)
kubectl delete <resource-type> <resource-name> -n <source-namespace>
```

## Practical Examples

### Creating a Development Environment
```bash
# Create development namespace
kubectl create namespace development

# Set context to development
kubectl config set-context --current --namespace=development

# Deploy application
kubectl create deployment myapp --image=myapp:dev
kubectl expose deployment myapp --port=80 --type=LoadBalancer
```

### Isolating Production Workloads
```yaml
# production-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "16"
    limits.memory: 32Gi
    pods: "50"
```

## Next Steps
- Learn about Cluster-level resources (not namespaced)
- Explore Kubernetes Network Policies
- Study Pod Security Policies
- Understand Service Accounts and RBAC in depth

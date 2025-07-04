# 12.6 Role-Based Access Control (RBAC)

## Introduction to RBAC

Role-Based Access Control (RBAC) is the primary authorization mechanism in Kubernetes, enabling fine-grained control over who can perform specific actions on resources within the cluster. RBAC is essential for:

- Implementing the principle of least privilege
- Securing cluster resources
- Managing team and application access
- Complying with security policies

## Core RBAC Components

### 1. Subjects
Entities that need access to the cluster:
- **Users**: Human operators or administrators
- **ServiceAccounts**: Processes running in pods
- **Groups**: Collections of users or service accounts

### 2. Resources
Kubernetes API objects that subjects can access:
- Pods, Deployments, Services, etc.
- Custom Resources (CRDs)
- API groups and non-resource URLs

### 3. Verbs
Actions that can be performed on resources:
- **Read Operations**: get, list, watch
- **Write Operations**: create, update, patch, delete
- **Privileged Operations**: use, bind, escalate

## RBAC API Objects

### 1. Role
Defines permissions within a specific namespace:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]  # Core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### 2. ClusterRole
Defines cluster-wide permissions or permissions that can be used across namespaces:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### 3. RoleBinding
Grants the permissions defined in a Role to a user or set of users within a namespace:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. ClusterRoleBinding
Grants cluster-wide permissions to a user or set of users:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-global
subjects:
- kind: Group
  name: system:authenticated
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

## Common RBAC Patterns

### 1. Namespace Administrator
```yaml
# Role: Full access to all resources in a namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: namespace-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

### 2. Read-Only Access
```yaml
# ClusterRole: Read-only access to all resources
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: global-reader
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

### 3. Specific Resource Access
```yaml
# Role: Access to specific resources and operations
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: deployment-manager
rules:
- apiGroups: ["apps", "extensions"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
```

## Aggregated ClusterRoles

ClusterRoles can be aggregated to combine multiple roles:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
# These rules will be added to the "monitoring" default role
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
```

## RBAC Best Practices

### 1. Least Privilege Principle
- Grant only the permissions that are absolutely necessary
- Start with minimal permissions and add more as needed

### 2. Use Groups Instead of Individual Users
- Makes management easier
- More scalable as teams grow
- Example: `developers@example.com` group instead of individual developer emails

### 3. Regular Audits
```bash
# Check cluster role bindings
kubectl get clusterrolebindings -o wide

# Check role bindings in all namespaces
kubectl get rolebindings --all-namespaces

# Check permissions for a specific user
kubectl auth can-i --list --as=system:serviceaccount:default:my-service-account
```

### 4. Use Meaningful Names
- Name roles based on their purpose
- Include scope in the name (e.g., `namespace-admin`, `cluster-reader`)

## Practical Examples

### 1. Creating a Developer Role
```yaml
# developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "jobs", "cronjobs"]
  verbs: ["create", "delete", "deletecollection", "get", "list", "patch", "update", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
```

### 2. Creating a CI/CD Service Account
```yaml
# ci-service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-pipeline
  namespace: ci-cd
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: ci-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-deployer-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: ci-pipeline
  namespace: ci-cd
roleRef:
  kind: Role
  name: ci-deployer
  apiGroup: rbac.authorization.k8s.io
```

## Troubleshooting RBAC

### Common Issues and Solutions

#### 1. "Forbidden" Errors
```bash
# Check if the user has the necessary permissions
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa

# Check role bindings
kubectl get rolebindings,clusterrolebindings --all-namespaces

# Check audit logs
kubectl get --raw /api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/api/v1/audit
```

#### 2. Missing Permissions
```bash
# Check what permissions a service account has
kubectl auth can-i --list --as=system:serviceaccount:default:my-sa

# Check cluster roles and bindings
kubectl get clusterrole,clusterrolebinding | grep my-role
```

#### 3. Debugging RBAC
```bash
# Check API discovery to verify resource names and API groups
kubectl api-resources

# Check API group for a specific resource
kubectl explain deployment.apiVersion

# Check effective permissions
kubectl auth can-i --list --as=system:serviceaccount:default:my-sa
```

## Advanced RBAC Patterns

### 1. Dynamic Admission Control with RBAC
- Use admission controllers to enforce RBAC policies
- Implement custom validation/mutation webhooks

### 2. Multi-tenancy with RBAC
- Use namespaces to separate tenants
- Implement tenant isolation using ResourceQuotas and NetworkPolicies
- Use RoleBindings to grant tenant-specific access

### 3. Self-Service Namespaces
```yaml
# Allow users to create namespaces and become admins in them
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-creator
rules:
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: namespace-creator-binding
subjects:
- kind: Group
  name: developers@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: namespace-creator
  apiGroup: rbac.authorization.k8s.io
```

## Summary

RBAC is a powerful and flexible system for controlling access to Kubernetes resources. By understanding and properly implementing RBAC, you can:

1. Secure your cluster with fine-grained access control
2. Implement the principle of least privilege
3. Enable team self-service while maintaining security boundaries
4. Comply with organizational and regulatory requirements

Remember to:
- Regularly audit your RBAC configurations
- Use groups instead of individual users when possible
- Follow naming conventions for roles and bindings
- Test permissions before applying them in production
- Document your RBAC policies and decisions

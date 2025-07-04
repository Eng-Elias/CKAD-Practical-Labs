# 12.5 API Access and ServiceAccounts

## Introduction to ServiceAccounts

ServiceAccounts in Kubernetes provide an identity for processes that run in Pods. They are essential for:

- Authenticating to the Kubernetes API server
- Managing permissions for Pods
- Controlling access to cluster resources
- Enabling secure communication between Pods and the API server

## ServiceAccount Basics

### Default ServiceAccount
- Every namespace has a default ServiceAccount
- Pods automatically use the default ServiceAccount if none is specified
- The default ServiceAccount has limited permissions

### Creating a ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: default
```

### Using a ServiceAccount in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-serviceaccount  # Specify the ServiceAccount
  containers:
  - name: my-container
    image: nginx
```

## ServiceAccount Tokens

### Automatic Token Mounting
- Kubernetes automatically mounts a ServiceAccount token at:
  ```
  /var/run/secrets/kubernetes.io/serviceaccount/token
  ```
- This token is used to authenticate to the API server

### Disabling Automatic Token Mounting
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  automountServiceAccountToken: false  # Disable automatic token mounting
  containers:
  - name: my-container
    image: nginx
```

### Manually Creating Tokens
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-serviceaccount-token
  annotations:
    kubernetes.io/service-account.name: my-serviceaccount
type: kubernetes.io/service-account-token
```

## RBAC with ServiceAccounts

### Role and RoleBinding Example
```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-viewer
  namespace: default
---
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-viewer
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Using ServiceAccounts in Applications

### Python Example
```python
from kubernetes import client, config

# Load the in-cluster config
config.load_incluster_config()

# Create API client
v1 = client.CoreV1Api()

# List pods in the default namespace
pods = v1.list_namespaced_pod(namespace="default")
for pod in pods.items:
    print(f"Pod: {pod.metadata.name}")
```

### Node.js Example
```javascript
const k8s = require('@kubernetes/client-node');

const kc = new k8s.KubeConfig();
kcloadFromDefault();

const k8sApi = kc.makeApiClient(k8s.CoreV1Api);

k8sApi.listNamespacedPod('default')
  .then((res) => {
    console.log('Pods:', res.body.items.map(pod => pod.metadata.name));
  })
  .catch((err) => {
    console.error('Error:', err);
  });
```

## Best Practices

### 1. Least Privilege
- Create dedicated ServiceAccounts for different applications
- Grant only the permissions needed
- Avoid using the default ServiceAccount

### 2. Security
- Rotate ServiceAccount tokens regularly
- Use RBAC to restrict access
- Monitor ServiceAccount usage

### 3. Naming Conventions
- Use descriptive names for ServiceAccounts
- Include the application name in the ServiceAccount name
- Use consistent naming across environments

## Troubleshooting

### Common Issues

#### 1. Permission Denied
```bash
# Check ServiceAccount exists
kubectl get serviceaccount my-serviceaccount -n my-namespace

# Check Role and RoleBinding
kubectl get role,rolebinding -n my-namespace

# Check effective permissions
kubectl auth can-i get pods --as=system:serviceaccount:my-namespace:my-serviceaccount
```

#### 2. Token Mount Issues
```bash
# Check if token is mounted
kubectl exec -it my-pod -- ls -la /var/run/secrets/kubernetes.io/serviceaccount/

# Check Pod spec
kubectl get pod my-pod -o yaml | grep serviceAccount
```

#### 3. API Server Authentication
```bash
# Check API server logs
kubectl logs -n kube-system kube-apiserver-controlplane

# Check audit logs if enabled
kubectl get --raw /api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/api/v1/audit
```

## Advanced Topics

### 1. TokenRequest API
- Generate time-bound tokens
- Better security than static tokens

```yaml
apiVersion: authentication.k8s.io/v1
kind: TokenRequest
metadata:
  name: my-token-request
spec:
  audiences: ["api"]
  expirationSeconds: 3600  # 1 hour
```

### 2. Projected ServiceAccount Tokens
- Mount tokens with specific attributes
- More secure than traditional tokens

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: kube-api-access
      mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      readOnly: true
  volumes:
  - name: kube-api-access
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          path: token
          expirationSeconds: 3600
          audience: api
      - configMap:
          name: kube-root-ca.crt
          items:
            - key: ca.crt
              path: ca.crt
      - downwardAPI:
          items:
            - path: namespace
              fieldRef:
                fieldPath: metadata.namespace
```

## Practical Example: Deploying an Application with Custom ServiceAccount

### 1. Create a Namespace
```bash
kubectl create namespace my-app
```

### 2. Create a ServiceAccount
```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: my-app
```

### 3. Create a Role
```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-app
  name: my-app-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
```

### 4. Create a RoleBinding
```yaml
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-role-binding
  namespace: my-app
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: my-app
roleRef:
  kind: Role
  name: my-app-role
  apiGroup: rbac.authorization.k8s.io
```

### 5. Deploy the Application
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: my-app-sa
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 8080
```

## Summary

ServiceAccounts are a fundamental part of Kubernetes security, providing identity for processes running in Pods. By following best practices and leveraging RBAC, you can create secure, least-privilege configurations for your applications. Always remember to:

1. Use dedicated ServiceAccounts for different applications
2. Apply the principle of least privilege
3. Regularly audit and rotate tokens
4. Monitor ServiceAccount usage
5. Use modern token management features like TokenRequest and projected volumes when possible

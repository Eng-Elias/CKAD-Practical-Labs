# 12.4 Authentication and Authorization

## Introduction to Kubernetes Security

Kubernetes provides a robust security model with multiple layers of protection:

1. **Authentication**: Verifying the identity of users and services
2. **Authorization**: Determining what actions an authenticated identity can perform
3. **Admission Control**: Validating and potentially modifying requests after authorization
4. **Network Policies**: Controlling pod-to-pod communication
5. **Secrets Management**: Securely storing sensitive information

## Authentication Methods

### 1. Client Certificates
- Most common authentication method
- Uses TLS client certificates
- Managed through the CertificateSigningRequest API

#### Creating a User Certificate
```bash
# Generate a private key
openssl genrsa -out jane.key 2048

# Create a certificate signing request (CSR)
openssl req -new -key jane.key \
  -out jane.csr \
  -subj "/CN=jane/O=developers"

# Get the base64-encoded CSR
cat jane.csr | base64 | tr -d '\n'
```

#### Create a CertificateSigningRequest
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane-developer
spec:
  request: $(cat jane.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # 1 day
  usages:
  - client auth
```

#### Approve and Retrieve the Certificate
```bash
# Approve the CSR
kubectl certificate approve jane-developer

# Get the signed certificate
kubectl get csr jane-developer -o jsonpath='{.status.certificate}' | base64 -d > jane.crt

# Set up kubeconfig
kubectl config set-credentials jane \
  --client-key=jane.key \
  --client-certificate=jane.crt \
  --embed-certs=true

# Set context
kubectl config set-context jane \
  --cluster=kubernetes \
  --user=jane
```

### 2. Bearer Tokens
- Used for service accounts and some authentication proxies
- Automatically mounted into pods at `/var/run/secrets/kubernetes.io/serviceaccount/token`

#### Using a Service Account Token
```bash
# Get the service account token
TOKEN=$(kubectl get secret $(kubectl get serviceaccount default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 --decode)

# Use the token for authentication
curl -k -H "Authorization: Bearer $TOKEN" https://<api-server>/api/v1/namespaces/default/pods
```

### 3. Authentication Proxy
- Used by cloud providers and some on-premise solutions
- Relies on HTTP headers for user identification
- Configured with `--requestheader-*` flags on the API server

## Authorization Methods

### 1. Role-Based Access Control (RBAC)
- Most commonly used authorization method
- Defines roles with specific permissions and binds them to users/service accounts

#### Example Role and RoleBinding
```yaml
# Role definition
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
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 2. Attribute-Based Access Control (ABAC)
- Less commonly used
- Uses policies defined in a file
- Requires API server restart to update policies

#### Example ABAC Policy
```json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "jane",
    "namespace": "default",
    "resource": "pods",
    "readonly": true
  }
}
```

### 3. Webhook Authorization
- Delegates authorization decisions to an external service
- Useful for custom authorization logic

#### Webhook Configuration Example
```yaml
apiVersion: v1
kind: Config
clusters:
  - name: name-of-remote-authz-service
    cluster:
      certificate-authority: /path/to/ca.pem
      server: https://authz.example.com/authorize

# Add to API server with:
# --authorization-webhook-config-file=webhook-config.yaml
# --authorization-mode=Webhook
```

## Practical Examples

### Checking Permissions
```bash
# Check if a user can create pods
kubectl auth can-i create pods --as=jane

# Check permissions in a specific namespace
kubectl auth can-i list pods --as=system:serviceaccount:default:default -n kube-system

# Check all permissions for the current user
kubectl auth can-i --list
```

### Creating a Service Account with Limited Access
```yaml
# Service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring
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
  name: monitoring-pod-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: monitoring
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Best Practices

### 1. Least Privilege
- Grant only the permissions that are absolutely necessary
- Regularly audit and clean up unused roles and bindings

### 2. Use Namespaces for Isolation
- Group related resources in namespaces
- Apply RBAC at the namespace level when appropriate

### 3. Regular Audits
```bash
# Find all role bindings
kubectl get rolebindings --all-namespaces

# Find all cluster role bindings
kubectl get clusterrolebindings

# Check effective permissions
kubectl auth can-i --list --as=system:serviceaccount:default:monitoring
```

### 4. Secure Service Accounts
- Avoid using the default service account
- Mount service account tokens only when needed
- Use dedicated service accounts for different applications

## Troubleshooting

### Common Issues

#### 1. "Forbidden" Errors
```bash
# Check if the user has the necessary permissions
kubectl auth can-i create pods --as=user@example.com

# Check role bindings
kubectl get rolebindings,clusterrolebindings --all-namespaces
```

#### 2. Certificate Issues
```bash
# Check certificate expiration
openssl x509 -in /path/to/cert.crt -noout -dates

# Verify certificate chain
openssl verify -CAfile /path/to/ca.crt /path/to/cert.crt
```

#### 3. Webhook Failures
```bash
# Check API server logs for webhook errors
kubectl logs -n kube-system kube-apiserver-controlplane

# Test the webhook endpoint manually
curl -k -X POST https://webhook-url/authorize -d @request.json
```

## Summary

Kubernetes provides a comprehensive security model with multiple authentication and authorization mechanisms. By understanding and properly configuring these components, you can ensure that your cluster remains secure while providing the necessary access to users and services. Always follow the principle of least privilege and regularly audit your security settings to maintain a strong security posture.

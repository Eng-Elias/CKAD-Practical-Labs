# 11.6 Understanding How Kubernetes Uses Secrets

## Secret Lifecycle in Kubernetes

### 1. Secret Creation
- Secrets can be created via `kubectl` or API
- Stored in etcd (base64 encoded by default)
- Can be created from literals, files, or YAML/JSON manifests

### 2. Secret Distribution
- Mounted into pods as:
  - Environment variables
  - Files in a volume
- Only sent to nodes that have pods requiring the secret
- Stored in `tmpfs` on the node (not written to disk)

### 3. Secret Usage
- Consumed by containers as files or environment variables
- Can be used by multiple pods across namespaces
- Automatically updated when the secret changes (with some caveats)

### 4. Secret Deletion
- Garbage collected when no longer referenced by any pods
- Can be manually deleted using `kubectl delete secret`

## How Kubernetes Manages Secrets

### 1. Storage in etcd
- By default, secrets are stored in etcd with base64 encoding
- Not encrypted unless etcd encryption is configured
- Access to etcd should be tightly controlled

### 2. On-Node Storage
- Secrets are sent to kubelet on nodes that need them
- Stored in `tmpfs` (in-memory filesystem)
- Never written to disk on the node
- Deleted when the pod is deleted

### 3. Secret Updates
- Secrets can be updated while pods are running
- Kubelet periodically checks for updates
- Updates may take time to propagate to all pods
- Some applications may need to be restarted to pick up changes

## Secret Access Control

### 1. Service Accounts
- Pods run with a service account
- Service accounts can be granted RBAC permissions to access secrets
- Default service account has limited permissions

### 2. RBAC for Secrets
- Use Role and RoleBinding to control access
- Example: Allow read-only access to specific secrets

```yaml
# secret-reader-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["db-credentials"]
  verbs: ["get", "watch", "list"]

# secret-reader-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: default
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

## Real-World Secret Usage Patterns

### 1. Database Credentials
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

### 2. TLS Certificates
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: |-
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
  tls.key: |-
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----
```

### 3. Docker Registry Authentication
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg
    image: private.registry.example.com/app:1.0
  imagePullSecrets:
  - name: regcred
```

## Secret Security Best Practices

### 1. Enable Encryption at Rest
```yaml
# encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: <base64-encoded-32-byte-key>
    - identity: {}
```

### 2. Use Network Policies
- Restrict pod-to-pod communication
- Prevent unauthorized access to pods with secrets

### 3. Regular Rotation
- Implement a process for rotating secrets
- Update all references when rotating
- Consider using external secret managers

## Monitoring and Auditing

### 1. Enable Audit Logging
```yaml
# audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]
```

### 2. Monitor Secret Access
```bash
# Check who accessed a secret
kubectl get --raw /api/v1/namespaces/default/secrets/my-secret | jq '.metadata.managedFields'

# Check audit logs for secret access
kubectl logs -n kube-system <kube-apiserver-pod> | grep -i secret
```

## Troubleshooting Secret Issues

### 1. Secret Not Found
```bash
# Check if secret exists
kubectl get secret my-secret

# Check events for errors
kubectl get events --field-selector involvedObject.name=my-secret
```

### 2. Permission Denied
```bash
# Check service account permissions
kubectl auth can-i get secret/my-secret --as=system:serviceaccount:default:default

# Check RBAC bindings
kubectl get rolebindings,clusterrolebindings --all-namespaces
```

### 3. Secret Not Updating
```bash
# Force update the deployment
kubectl rollout restart deployment/myapp

# Check secret version in pod
kubectl exec -it mypod -- cat /etc/secret-volume/..data/username
```

## Advanced Secret Management

### 1. External Secret Management
- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

### 2. Sealed Secrets
```bash
# Install kubeseal
kubeseal --fetch-cert \
  --controller-namespace=kube-system \
  > pub-cert.pem

# Create a sealed secret
kubectl create secret generic mysecret \
  --dry-run=client \
  --from-literal=foo=bar \
  -o json | kubeseal \
  --cert pub-cert.pem \
  > mysealedsecret.json
```

### 3. CSI Secret Store
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: my-secret-provider
spec:
  provider: azure
  parameters:
    usePodIdentity: "true"
    keyvaultName: "my-keyvault"
    objects: |
      array:
        - |
          objectName: my-secret
          objectType: secret
    tenantId: "my-tenant-id"
```

## Summary

Understanding how Kubernetes manages secrets is crucial for building secure applications. By following best practices for secret management, implementing proper access controls, and leveraging advanced features like encryption and external secret managers, you can ensure that sensitive information remains protected throughout its lifecycle in your Kubernetes cluster.

# 11.5 Understanding Secrets

## Introduction to Kubernetes Secrets

Secrets in Kubernetes are designed to store and manage sensitive information such as passwords, OAuth tokens, and SSH keys. They provide a more secure way to handle sensitive data compared to storing them in Pod specifications or container images.

## Types of Secrets

### 1. Opaque Secrets
- Generic secret type
- Default type when creating secrets from files or literals
- Base64 encoded key-value pairs

### 2. Service Account Token Secrets
- Automatically created for service accounts
- Used for pod authentication with the API server

### 3. Docker Registry Secrets
- Store credentials for container registry authentication
- Used when pulling images from private registries

### 4. TLS Secrets
- Store TLS certificates and keys
- Used for securing Ingress resources

## Creating Secrets

### 1. From Literal Values

```bash
# Create a generic secret
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='S!B\$dopsz'  # Use single quotes to preserve special characters

# View the secret (encoded)
kubectl get secret db-credentials -o yaml

# Decode a secret value
echo 'UyFCXGRvc3p6' | base64 --decode  # Outputs: S!B\$dopsz
```

### 2. From Files

```bash
# Create files with sensitive data
echo -n 'admin' > ./username.txt
echo -n 'S!B\$dopsz' > ./password.txt

# Create secret from files
kubectl create secret generic db-credentials \
  --from-file=./username.txt \
  --from-file=./password.txt
```

### 3. Using YAML Definition

```yaml
# db-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=  # echo -n 'admin' | base64
  password: UyFCXGRvc3p6  # echo -n 'S!B\$dopsz' | base64
```

## Using Secrets in Pods

### 1. As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: password
```

### 2. Mounting as Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    volumeMounts:
    - name: creds
      mountPath: "/etc/creds"
      readOnly: true
  volumes:
  - name: creds
    secret:
      secretName: db-credentials
      # Optional: Change file permissions (default is 0644)
      defaultMode: 0400
      # Optional: Specify which keys to include
      items:
      - key: username
        path: my-username
      - key: password
        path: my-password
```

## Best Practices for Managing Secrets

### 1. Secret Security
- **Avoid** storing secrets in version control
- Use external secret management systems (HashiCorp Vault, AWS Secrets Manager) for production
- Enable encryption at rest for etcd
- Use RBAC to restrict access to secrets

### 2. Secret Rotation
- Implement automated secret rotation
- Use tools like External Secrets Operator or Sealed Secrets
- Update deployments when secrets change

### 3. Secret Naming Conventions
- Use consistent naming (e.g., `<app>-<environment>-<purpose>`)
- Include version information (e.g., `db-credentials-v1`)

## Example: MySQL with Secrets

### 1. Create MySQL Secret

```yaml
# mysql-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  # echo -n 'root' | base64
  mysql-root-password: cm9vdA==
  # echo -n 'dbuser' | base64
  mysql-username: ZGJ1c2Vy
  # echo -n 'dbpass' | base64
  mysql-password: ZGJwYXNz
```

### 2. Create MySQL Deployment

```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-credentials
              key: mysql-root-password
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-credentials
              key: mysql-username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-credentials
              key: mysql-password
        ports:
        - containerPort: 3306
```

## Troubleshooting Secrets

### Common Issues

1. **Permission Denied**
   - Check if the pod's service account has permission to access the secret
   - Verify the secret exists in the same namespace as the pod

2. **Secret Not Found**
   ```bash
   # Check if secret exists
   kubectl get secret my-secret
   
   # Describe pod for errors
   kubectl describe pod my-pod
   ```

3. **Incorrect Secret Values**
   - Verify the secret data is properly base64 encoded
   - Check for trailing newlines in the secret values

4. **Mounting Issues**
   ```bash
   # Check mounted secrets in a running pod
   kubectl exec -it my-pod -- ls -la /etc/secret-volume
   
   # View environment variables
   kubectl exec -it my-pod -- env | grep SECRET_
   ```

## Security Considerations

1. **etcd Encryption**
   - By default, secrets are stored in etcd in base64 encoding (not encryption)
   - Enable encryption at rest for production clusters

2. **Access Control**
   - Use RBAC to restrict access to secrets
   - Limit who can create and view secrets

3. **Secret Rotation**
   - Implement a process for rotating secrets
   - Update all references when rotating secrets

4. **Audit Logging**
   - Enable audit logging for secret access
   - Monitor for unauthorized access attempts

## Summary

Kubernetes Secrets provide a secure way to manage sensitive information in your cluster. By following best practices for creation, management, and security, you can effectively protect your sensitive data while making it available to your applications when needed.

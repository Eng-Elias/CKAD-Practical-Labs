# 11.7 Configuring Applications to Use Secrets

## Introduction to Application Configuration with Secrets

This section covers practical patterns for configuring applications to use Kubernetes Secrets. We'll explore various methods to inject secrets into your applications securely and efficiently.

## Environment Variables from Secrets

### 1. Individual Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

### 2. Environment Variables from Multiple Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multiple-secrets-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: api-credentials
          key: key
```

## Mounting Secrets as Files

### 1. Basic Secret Volume Mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-file-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
```

### 2. Mounting Specific Secret Keys as Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: specific-secret-keys-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/ssl/certs
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: tls-secret
      items:
      - key: tls.crt
        path: tls.crt
      - key: tls.key
        path: tls.key
```

## Using Secrets in Deployments

### 1. Deployment with Environment Variables

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        env:
        - name: DB_CONNECTION_STRING
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: connection-string
        ports:
        - containerPort: 8080
```

### 2. Deployment with Volume Mounts

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:alpine
        volumeMounts:
        - name: tls-secret
          mountPath: /etc/nginx/ssl
          readOnly: true
      volumes:
      - name: tls-secret
        secret:
          secretName: tls-secret
```

## Advanced Secret Usage Patterns

### 1. Using SubPath with Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-subpath-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /app/config/credentials.json
      subPath: credentials.json
  volumes:
  - name: secret-volume
    secret:
      secretName: app-credentials
      items:
      - key: credentials.json
        path: credentials.json
```

### 2. Setting File Permissions

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-permissions-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
      defaultMode: 0400  # Read-only for owner
      items:
      - key: config.json
        path: config.json
        mode: 0600  # Read-write for owner
```

## Using Secrets with Environment Variables

### 1. Environment Variables with Default Values

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-default-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: DB_HOST
      value: "localhost"  # Default value
    - name: DB_PORT
      value: "5432"       # Default value
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
          optional: true  # Makes this environment variable optional
```

### 2. Environment Variables from ConfigMap and Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-and-secret-pod
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: environment
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

## Best Practices for Application Configuration

### 1. Security
- Never log or expose secret values in application logs
- Use file permissions to restrict access to mounted secrets
- Consider using a secret management service for production

### 2. Configuration Management
- Use ConfigMaps for non-sensitive configuration
- Use Secrets for sensitive data
- Consider using a configuration library that supports hot reloading

### 3. Deployment
- Use Kubernetes RBAC to control access to secrets
- Consider using sealed secrets or external secret managers
- Rotate secrets regularly

## Example: Complete Application with Secrets

### 1. Create a Secret

```yaml
# app-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # echo -n 'postgres://user:password@db:5432/mydb' | base64
  database-url: cG9zdGdyZXM6Ly91c2VyOnBhc3N3b3JkQGRiOjU0MzIvbXlkYg==
  # echo -n 'your-secret-key-here' | base64
  api-key: eW91ci1zZWNyZXQta2V5LWhlcmU=
```

### 2. Create a ConfigMap

```yaml
# app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.ini: |
    [database]
    max_connections = 100
    timeout = 30
    
    [server]
    port = 8080
    debug = false
```

### 3. Create a Deployment

```yaml
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: api-key
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
          items:
          - key: app.ini
            path: app.ini
```

## Troubleshooting

### Common Issues

1. **Secret Not Found**
   ```bash
   # Check if secret exists
   kubectl get secret my-secret
   
   # Describe pod for errors
   kubectl describe pod my-pod
   ```

2. **Permission Denied**
   ```bash
   # Check pod logs for permission errors
   kubectl logs my-pod
   
   # Check file permissions in the container
   kubectl exec -it my-pod -- ls -la /path/to/secret
   ```

3. **Environment Variable Not Set**
   ```bash
   # Check environment variables in the container
   kubectl exec -it my-pod -- env | grep DB_
   ```

4. **Configuration File Not Found**
   ```bash
   # Check mounted volumes
   kubectl exec -it my-pod -- ls -la /path/to/config
   ```

## Summary

Configuring applications to use Kubernetes Secrets involves several patterns and best practices. By following the examples and guidelines in this section, you can securely manage sensitive information in your Kubernetes applications while maintaining flexibility and security.

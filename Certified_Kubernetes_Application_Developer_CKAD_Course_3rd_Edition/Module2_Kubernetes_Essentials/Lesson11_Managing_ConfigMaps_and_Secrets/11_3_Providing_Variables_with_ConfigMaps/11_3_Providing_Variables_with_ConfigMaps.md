# 11.3 Providing Variables with ConfigMaps

## Introduction to ConfigMaps

ConfigMaps in Kubernetes provide a way to decouple configuration artifacts from container images, making applications more portable. They store configuration data as key-value pairs that can be consumed by pods or other system components.

## Creating ConfigMaps

### 1. From Literal Values

```bash
# Create ConfigMap with literal values
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod

# Verify creation
kubectl get configmap app-config -o yaml
```

### 2. From Environment Files

```bash
# Create a .env file
echo "APP_COLOR=blue
APP_MODE=prod" > app.env

# Create ConfigMap from file
kubectl create configmap app-config --from-env-file=app.env
```

### 3. From Configuration Files

```bash
# Create a config file
cat > config.json <<EOF
{
  "database": {
    "host": "db-service",
    "port": 3306
  }
}
EOF

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.json
```

### 4. Using YAML Definition

```yaml
# app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: "blue"
  APP_MODE: "prod"
  config.json: |
    {
      "database": {
        "host": "db-service",
        "port": 3306
      }
    }
```

## Using ConfigMaps in Pods

### 1. As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ["/bin/sh", "-c", "env"]
      envFrom:
      - configMapRef:
          name: app-config
```

### 2. As Individual Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: busybox
      command: ["/bin/sh", "-c", "echo $APP_COLOR && echo $APP_MODE"]
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
```

### 3. As Volume Mounts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: nginx
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
        items:
        - key: config.json
          path: config.json
```

## Updating ConfigMaps

### 1. Update ConfigMap

```bash
# Edit ConfigMap directly
kubectl edit configmap app-config

# Or update from file
kubectl create configmap app-config --from-file=config.json -o yaml --dry-run=client | kubectl apply -f -
```

### 2. Trigger Pod Update

Pods don't automatically detect ConfigMap changes. To update pods:

```bash
# Method 1: Restart the deployment
kubectl rollout restart deployment/myapp

# Method 2: Patch the deployment to force update
kubectl patch deployment myapp -p '{"spec":{"template":{"metadata":{"annotations":{"last-updated":"'$(date +%s)'"}}}}}'
```

## Best Practices

1. **Immutable ConfigMaps**
   - Use `immutable: true` for ConfigMaps that don't change
   - Improves performance and security

2. **Namespace Awareness**
   - ConfigMaps are namespaced
   - Ensure pods and ConfigMaps are in the same namespace

3. **Size Limitations**
   - ConfigMaps have a 1MB size limit
   - For larger configurations, consider mounting as volumes

4. **Sensitive Data**
   - Never store sensitive data in ConfigMaps
   - Use Secrets for sensitive information

## Example: Complete Application

### 1. Create ConfigMap

```yaml
# frontend-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config
data:
  APP_NAME: "My Awesome App"
  THEME_COLOR: "#007bff"
  FEATURE_FLAGS: |
    dark_mode=true
    notifications=true
    analytics=false
```

### 2. Create Deployment

```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:alpine
        ports:
        - containerPort: 80
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: frontend-config
              key: APP_NAME
        - name: THEME_COLOR
          valueFrom:
            configMapKeyRef:
              name: frontend-config
              key: THEME_COLOR
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: config-volume
        configMap:
          name: frontend-config
          items:
          - key: FEATURE_FLAGS
            path: feature_flags.conf
```

## Troubleshooting

### Common Issues

1. **ConfigMap Not Found**
   ```bash
   # Verify ConfigMap exists
   kubectl get configmap app-config
   
   # Check events
   kubectl describe pod/myapp-pod
   ```

2. **Permission Denied**
   - Ensure the pod's service account has permission to read the ConfigMap

3. **ConfigMap Not Updated**
   - Remember to restart pods after updating ConfigMap
   - Check if the ConfigMap is mounted as a volume (updates are eventually consistent)

4. **Invalid YAML/JSON**
   - Validate your ConfigMap data before applying
   - Use `kubectl create configmap --dry-run=client -o yaml` to test

## Summary

ConfigMaps provide a powerful way to manage configuration data in Kubernetes. By following these patterns and best practices, you can create more maintainable and portable applications.

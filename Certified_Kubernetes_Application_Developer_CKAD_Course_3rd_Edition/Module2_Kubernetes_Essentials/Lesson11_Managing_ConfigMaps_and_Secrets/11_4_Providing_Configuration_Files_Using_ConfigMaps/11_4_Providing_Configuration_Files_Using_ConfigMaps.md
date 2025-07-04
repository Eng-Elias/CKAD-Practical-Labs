# 11.4 Providing Configuration Files Using ConfigMaps

## Introduction

ConfigMaps in Kubernetes can be used to provide entire configuration files to containers, making it easy to manage application configuration without rebuilding container images. This approach is particularly useful for:

- Configuration files that are too large for environment variables
- Applications that expect configuration in specific file formats (JSON, YAML, INI, etc.)
- Configuration that needs to be mounted at specific paths in the container

## Creating ConfigMaps from Files

### 1. From a Single File

```bash
# Create a sample configuration file
cat > nginx.conf <<'EOF'
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF

# Create ConfigMap from file
kubectl create configmap nginx-config --from-file=nginx.conf

# Verify the ConfigMap
kubectl get configmap nginx-config -o yaml
```

### 2. From a Directory

```bash
# Create a directory with multiple config files
mkdir configs

echo "debug=true" > configs/application.properties
echo "log.level=INFO" >> configs/application.properties

# Create ConfigMap from directory
kubectl create configmap app-config --from-file=configs/
```

### 3. With Custom Keys

```bash
# Create ConfigMap with custom key names
kubectl create configmap app-config \
  --from-file=my-nginx-config=nginx.conf \
  --from-file=app-properties=configs/application.properties
```

## Mounting ConfigMaps as Volumes

### Basic Volume Mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: default.conf
```

### Mounting Multiple Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
  - name: demo
    image: alpine
    command: ["sh", "-c", "ls -l /etc/config/ && cat /etc/config/*"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: nginx.conf
        path: nginx/nginx.conf
      - key: application.properties
        path: app/application.properties
```

## Advanced ConfigMap Features

### 1. Setting File Permissions

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
    defaultMode: 0600  # Sets permissions to rw-------
    items:
    - key: nginx.conf
      path: nginx/nginx.conf
      mode: 0644  # Override default mode for specific file
```

### 2. Using SubPath

Mount a single file from a ConfigMap without mounting the entire directory:

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/nginx/nginx.conf
  subPath: nginx.conf
```

### 3. Immutable ConfigMaps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true  # Makes the ConfigMap immutable
data:
  app.properties: |
    cache.enabled=true
    log.level=INFO
```

## Real-World Example: WordPress with ConfigMap

### 1. Create WordPress Configuration

```yaml
# wordpress-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
data:
  wp-config.php: |
    <?php
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wordpress');
    define('DB_PASSWORD', 'password');
    define('DB_HOST', 'mysql:3306');
    define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST']);
    define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST']);
    
    if (!defined('ABSPATH')) {
        define('ABSPATH', dirname(__FILE__) . '/');
    }
    
    require_once(ABSPATH . 'wp-settings.php');
```

### 2. Create WordPress Deployment

```yaml
# wordpress-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:php7.4-apache
        ports:
        - containerPort: 80
        volumeMounts:
        - name: wordpress-config
          mountPath: /var/www/html/wp-config.php
          subPath: wp-config.php
      volumes:
      - name: wordpress-config
        configMap:
          name: wordpress-config
          items:
          - key: wp-config.php
            path: wp-config.php
```

## Best Practices

1. **Organization**
   - Group related configuration files in the same ConfigMap
   - Use meaningful names for ConfigMaps and keys

2. **Versioning**
   - Include version information in ConfigMap names
   - Example: `app-config-v1`, `app-config-v2`

3. **Updates**
   - Be aware that ConfigMap updates are eventually consistent
   - Consider using a sidecar like `Reloader` to watch for changes

4. **Sensitive Data**
   - Never store sensitive information in ConfigMaps
   - Use Secrets for sensitive data

## Troubleshooting

### Common Issues

1. **File Permissions**
   - Ensure the container user has permission to read the mounted files
   - Check file permissions using `kubectl exec`

2. **File Not Found**
   - Verify the mount path and subPath are correct
   - Check if the ConfigMap key matches the expected filename

3. **Configuration Not Updating**
   - Remember that volume-mounted ConfigMaps update eventually
   - Use `kubectl rollout restart deployment/<name>` to force an update

4. **Invalid Configuration**
   - Validate configuration files before applying
   - Check container logs for parsing errors

## Summary

Using ConfigMaps to provide configuration files is a powerful pattern in Kubernetes that enables:
- Clean separation of configuration from container images
- Easy updates without rebuilding containers
- Consistent configuration across environments
- Version control for configuration changes

By following these patterns and best practices, you can effectively manage application configuration in a Kubernetes-native way.

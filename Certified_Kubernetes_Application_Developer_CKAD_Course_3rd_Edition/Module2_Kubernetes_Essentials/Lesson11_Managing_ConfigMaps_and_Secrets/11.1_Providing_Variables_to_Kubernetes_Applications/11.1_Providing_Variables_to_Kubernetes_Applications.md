# 11.1 Providing Variables to Kubernetes Applications

## Introduction

In Kubernetes, providing environment variables to applications can be done in several ways. This section covers how to set and manage environment variables in pods and deployments.

## Setting Environment Variables in Deployments

### Using `kubectl set env`

To set environment variables in a running deployment:

```bash
# Set a single environment variable
kubectl set env deployment/mydb MYSQL_ROOT_PASSWORD=password

# Set multiple environment variables
kubectl set env deployment/mydb \
  MYSQL_ROOT_PASSWORD=password \
  MYSQL_DATABASE=mydb
```

### Creating a Deployment with Environment Variables

When creating a deployment, you can specify environment variables directly in the YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydb
spec:
  selector:
    matchLabels:
      app: mydb
  template:
    metadata:
      labels:
        app: mydb
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        - name: MYSQL_DATABASE
          value: "mydb"
```

## Common Issues and Solutions

### Variable Not Being Set

If your environment variables aren't being set as expected:

1. Check the deployment status:
   ```bash
   kubectl describe deployment/mydb
   ```

2. Check the pod logs:
   ```bash
   kubectl logs deployment/mydb
   ```

3. Verify the environment variables in a running pod:
   ```bash
   kubectl exec -it <pod-name> -- env | grep MYSQL
   ```

### Updating Environment Variables

To update environment variables in a deployment:

```bash
# Update a single variable
kubectl set env deployment/mydb MYSQL_ROOT_PASSWORD=newpassword

# Remove an environment variable
kubectl set env deployment/mydb MYSQL_DATABASE-
```

## Best Practices

1. **Use ConfigMaps for Configuration**
   - Store non-sensitive configuration in ConfigMaps
   - Reference ConfigMap values in your deployment

2. **Use Secrets for Sensitive Data**
   - Never store passwords or API keys directly in deployment YAML
   - Use Kubernetes Secrets for sensitive information

3. **Document Your Variables**
   - Maintain a list of required environment variables
   - Document the purpose and expected values

## Example: MariaDB Deployment

Here's a complete example of deploying MariaDB with environment variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:10.5
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "my-secret-pw"
        - name: MYSQL_DATABASE
          value: "mydb"
        - name: MYSQL_USER
          value: "user"
        - name: MYSQL_PASSWORD
          value: "user-password"
        ports:
        - containerPort: 3306
          name: mysql
```

## Troubleshooting

### Checking Environment Variables

To verify environment variables in a running pod:

```bash
# List all environment variables
kubectl exec -it <pod-name> -- env

# Check specific variable
kubectl exec -it <pod-name> -- printenv MYSQL_ROOT_PASSWORD
```

### Common Errors

1. **Permission Denied**
   - Ensure the container user has permissions to read the environment variables

2. **Variable Not Found**
   - Check for typos in variable names
   - Verify the variable is defined in the deployment

3. **Application Not Picking Up Changes**
   - Some applications only read environment variables at startup
   - Consider restarting the pod if variables were updated

## Next Steps

In the next section, we'll explore how to use ConfigMaps to manage configuration data more effectively.

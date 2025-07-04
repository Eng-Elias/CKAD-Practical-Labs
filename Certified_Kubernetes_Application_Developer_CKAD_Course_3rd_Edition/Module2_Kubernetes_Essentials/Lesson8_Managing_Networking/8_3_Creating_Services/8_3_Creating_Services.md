# 8.3 Creating Services

## Introduction to Service Creation

Kubernetes Services can be created using either imperative or declarative approaches. This section covers both methods with practical examples.

## Service Ports Explained

### Key Port Types
1. **port**
   - The port the service listens on
   - Used to access the service within the cluster
   - Example: `port: 80`

2. **targetPort**
   - The port on the Pods where the application is running
   - Defaults to the same value as `port` if not specified
   - Example: `targetPort: 8080`

3. **nodePort** (for NodePort services)
   - The port on each node where the service is exposed
   - Default range: 30000-32767
   - Example: `nodePort: 31000`

## Creating Services

### 1. Using `kubectl expose` (Imperative)

```bash
# Create a deployment
kubectl create deployment nginx --image=nginx

# Expose as ClusterIP (default)
kubectl expose deployment nginx --port=80 --target-port=80

# Expose as NodePort
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort

# Expose with a specific NodePort
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort --name=nginx-nodeport
```

### 2. Using YAML Manifest (Declarative)

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80        # Service port
      targetPort: 80  # Container port
      nodePort: 30007 # Optional: specify NodePort
```

Apply the manifest:
```bash
kubectl apply -f nginx-service.yaml
```

## Verifying Services

### Basic Commands

```bash
# List all services
kubectl get services

# Get detailed information
kubectl describe service nginx-service

# View service endpoints
kubectl get endpoints nginx-service

# Get service URL in Minikube
minikube service nginx-service --url
```

### Testing Service Connectivity

```bash
# From outside the cluster (NodePort)
curl $(minikube ip):<nodePort>

# From inside the cluster
kubectl run -it --rm --image=curlimages/curl curl -- sh
curl http://nginx-service
```

## Advanced Service Configurations

### Headless Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None  # Makes it headless
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### ExternalName Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: my.database.example.com
```

## Service Updates and Maintenance

### Updating a Service

```bash
# Method 1: Edit directly
kubectl edit svc nginx-service

# Method 2: Apply updated YAML
kubectl apply -f updated-service.yaml
```

### Deleting a Service

```bash
# Delete by name
kubectl delete service nginx-service

# Delete by label
kubectl delete service -l app=nginx
```

## Troubleshooting Common Issues

### Service Not Accessible
1. Check if Pods are running and ready
   ```bash
   kubectl get pods -l app=nginx
   ```

2. Verify service selector matches Pod labels
   ```bash
   kubectl get pods --show-labels
   kubectl describe svc nginx-service
   ```

3. Check service endpoints
   ```bash
   kubectl get endpoints nginx-service
   ```

### Port Forwarding for Debugging

```bash
# Forward local port to service
kubectl port-forward svc/nginx-service 8080:80

# Test locally
curl http://localhost:8080
```

## Best Practices

1. **Naming Conventions**
   - Use lowercase names with hyphens
   - Be descriptive (e.g., `user-service`, `payment-gateway`)
   - Include environment if needed (e.g., `user-service-staging`)

2. **Port Management**
   - Document port usage
   - Avoid hardcoding NodePort values when possible
   - Use descriptive port names for multiple ports

3. **Resource Management**
   - Set appropriate resource requests/limits
   - Monitor service performance
   - Clean up unused services

## Next Steps
In the next section, we'll explore how to use services in microservices architectures.

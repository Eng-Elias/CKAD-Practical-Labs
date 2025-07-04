# 9.2 Configuring the Minikube Ingress Controller

## Introduction

Minikube provides a simple way to enable and use the Ingress controller through its add-ons system. This section covers how to set up and verify the NGINX Ingress controller in Minikube.

## Enabling the Ingress Add-on

### 1. Check Available Add-ons
```bash
minikube addons list
```

### 2. Enable the Ingress Controller
```bash
minikube addons enable ingress
```

### 3. Verify the Installation
```bash
kubectl get pods -n ingress-nginx
```

## Understanding the Ingress Controller Components

### 1. Namespace
```bash
kubectl get ns | grep ingress
```

### 2. Deployments
```bash
kubectl get deployments -n ingress-nginx
```

### 3. Services
```bash
kubectl get services -n ingress-nginx
```

## Verifying the Ingress Controller

### 1. Check the Ingress Controller Pod
```bash
kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### 2. Check the Service
```bash
kubectl get svc -n ingress-nginx
```

### 3. Check Logs
```bash
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --follow
```

## Basic Ingress Example

### 1. Create a Test Deployment
```yaml
# test-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-minikube
spec:
  selector:
    matchLabels:
      app: hello-minikube
  template:
    metadata:
      labels:
        app: hello-minikube
    spec:
      containers:
      - name: hello-minikube
        image: k8s.gcr.io/echoserver:1.10
        ports:
        - containerPort: 8080
```

### 2. Create a Service
```yaml
# test-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-minikube
spec:
  selector:
    app: hello-minikube
  ports:
  - port: 80
    targetPort: 8080
```

### 3. Create an Ingress Resource
```yaml
# test-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello-minikube
            port:
              number: 80
```

### 4. Apply the Configuration
```bash
kubectl apply -f test-deployment.yaml
kubectl apply -f test-service.yaml
kubectl apply -f test-ingress.yaml
```

### 5. Test the Ingress
```bash
# Get the Minikube IP
minikube ip

# Test the endpoint
curl $(minikube ip)/hello
```

## Common Issues and Solutions

### 1. Ingress Controller Not Starting
```bash
# Check pod status
kubectl get pods -n ingress-nginx

# Check logs
kubectl logs -n ingress-nginx <pod-name>

# Delete and recreate the add-on
minikube addons disable ingress
minikube addons enable ingress
```

### 2. 404 Errors
- Verify the service name and port in the Ingress resource
- Check if the service has endpoints: `kubectl get endpoints <service-name>`
- Verify the path and pathType in the Ingress rules

### 3. DNS Resolution Issues
```bash
# On Linux/macOS
sudo echo "$(minikube ip) example.com" | sudo tee -a /etc/hosts

# On Windows (Run as Administrator)
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "$(minikube ip) example.com"
```

## Advanced Configuration

### 1. Customizing the Ingress Controller
```bash
# View the current configuration
kubectl get configmap -n ingress-nginx

# Edit the config map
kubectl edit configmap nginx-configuration -n ingress-nginx
```

### 2. Enabling SSL/TLS
```yaml
# tls-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-minikube
            port:
              number: 80
```

### 3. Setting Up a Default Backend
```yaml
# default-backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: default-http-backend
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: default-http-backend
  template:
    metadata:
      labels:
        app: default-http-backend
    spec:
      containers:
      - name: default-http-backend
        image: k8s.gcr.io/defaultbackend-amd64:1.5
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 10m
            memory: 20Mi
          requests:
            cpu: 10m
            memory: 20Mi
```

## Best Practices

1. **Resource Management**
   - Set appropriate resource requests and limits for the Ingress controller
   - Monitor resource usage

2. **High Availability**
   - Run multiple replicas of the Ingress controller
   - Use pod anti-affinity rules

3. **Security**
   - Restrict access using network policies
   - Regularly update the Ingress controller
   - Implement proper TLS configuration

4. **Monitoring**
   - Set up monitoring for the Ingress controller
   - Monitor request rates and error rates

## Next Steps
In the next section, we'll explore how to create and manage Ingress resources to route traffic to your services.

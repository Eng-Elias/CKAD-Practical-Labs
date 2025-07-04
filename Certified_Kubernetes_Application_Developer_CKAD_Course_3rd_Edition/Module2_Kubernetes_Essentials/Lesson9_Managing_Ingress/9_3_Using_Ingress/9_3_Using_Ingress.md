# 9.3 Using Ingress

## Introduction to Ingress Resources

In this section, we'll explore how to create and manage Ingress resources to route external traffic to your services.

## Basic Ingress Example

### 1. Create a Test Deployment and Service

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 2. Create a Simple Ingress Resource

```yaml
# simple-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```

### 3. Apply the Configuration

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f simple-ingress.yaml
```

## Verifying the Ingress

### 1. Check Ingress Status

```bash
kubectl get ingress
```

### 2. Test Access

```bash
# Get the Minikube IP
MINIKUBE_IP=$(minikube ip)

# Test access
curl $MINIKUBE_IP
```

## Path-Based Routing

### Example: Multiple Paths to Different Services

```yaml
# path-based-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

## Name-Based Virtual Hosting

### Example: Multiple Hosts

```yaml
# name-based-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-based-ingress
spec:
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

## Using Annotations

### Common Annotations

```yaml
# annotated-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
    nginx.ingress.kubernetes.io/app-root: /app1
spec:
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

## TLS Configuration

### 1. Create a TLS Secret

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=example.com/O=example"

kubectl create secret tls example-tls --key tls.key --cert tls.crt
```

### 2. Create TLS Ingress

```yaml
# tls-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
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
            name: app-service
            port:
              number: 80
```

## Troubleshooting Ingress

### Common Issues and Solutions

1. **Ingress Controller Not Found**
   ```bash
   # Check if the Ingress controller is running
   kubectl get pods -n ingress-nginx
   ```

2. **Service Not Found**
   ```bash
   # Check services
   kubectl get svc
   
   # Check endpoints
   kubectl get endpoints
   ```

3. **404 Errors**
   - Verify the path and pathType in the Ingress resource
   - Check if the backend service has endpoints
   - Verify the service port matches the target port

4. **TLS/SSL Issues**
   ```bash
   # Check TLS secret
   kubectl get secret example-tls
   
   # Check Ingress events
   kubectl describe ingress tls-ingress
   ```

## Best Practices

1. **Use Meaningful Names**
   - Choose descriptive names for Ingress resources
   - Include environment in the name if applicable (e.g., `-prod`, `-staging`)

2. **Path Configuration**
   - Use `pathType: Prefix` for most cases
   - Be specific with paths to avoid conflicts
   - Consider using `rewrite-target` for complex routing

3. **Security**
   - Always use TLS in production
   - Implement proper network policies
   - Consider using WAF (Web Application Firewall)

4. **Monitoring and Logging**
   - Set up monitoring for your Ingress controller
   - Enable access logs
   - Monitor error rates and response times

## Next Steps
In the next section, we'll dive deeper into configuring Ingress rules and explore more advanced routing scenarios.

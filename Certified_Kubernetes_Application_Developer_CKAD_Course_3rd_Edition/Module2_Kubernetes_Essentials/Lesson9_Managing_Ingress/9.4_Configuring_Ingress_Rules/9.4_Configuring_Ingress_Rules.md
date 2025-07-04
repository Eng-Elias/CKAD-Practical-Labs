# 9.4 Configuring Ingress Rules

## Understanding Ingress Rules

Ingress rules define how external HTTP/HTTPS traffic is routed to services within your cluster. Each rule consists of:

- **Host**: Optional domain name (e.g., example.com)
- **Path**: URL path (e.g., /api)
- **Backend**: Service and port to route traffic to
- **Path Type**: How paths are matched (Exact, Prefix, or ImplementationSpecific)

## Path Types

### 1. Exact
Matches the URL path exactly (case-sensitive).

### 2. Prefix
Matches based on a URL path prefix (e.g., /api matches /api/v1, /api/v2).

### 3. ImplementationSpecific
Interpretation depends on the IngressClass.

## Rule Configuration Examples

### Basic Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-routing
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

### Name-Based Virtual Hosting

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-based-hosting
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

## Advanced Routing Scenarios

### URL Rewriting

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: url-rewrite
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - http:
      paths:
      - path: /something(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: http-svc
            port:
              number: 80
```

### SSL/TLS Termination

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example
spec:
  tls:
  - hosts:
    - sslexample.example.com
    secretName: example-tls
  rules:
  - host: sslexample.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```

## Default Backend

A default backend handles requests that don't match any rules.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend
spec:
  defaultBackend:
    service:
      name: default-http-backend
      port:
        number: 80
  rules:
  - host: example.com
    http:
      paths:
      - path: /test
        pathType: Prefix
        backend:
          service:
            name: test-service
            port:
              number: 80
```

## Advanced Configuration Options

### Rate Limiting

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limited
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-rpm: "1000"
    nginx.ingress.kubernetes.io/limit-burst-multiplier: "5"
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### CORS Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cors-enabled
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization"
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

## Best Practices

1. **Use Meaningful Names**
   - Choose descriptive names for your Ingress resources
   - Include environment in the name if applicable

2. **Path Configuration**
   - Be specific with paths to avoid conflicts
   - Use `pathType: Prefix` for most cases
   - Consider using `rewrite-target` for complex routing

3. **Host Configuration**
   - Always specify hosts in production
   - Use wildcard certificates when possible

4. **Security**
   - Always use TLS in production
   - Implement proper network policies
   - Consider using WAF (Web Application Firewall)

5. **Monitoring and Logging**
   - Set up monitoring for your Ingress controller
   - Enable access logs
   - Monitor error rates and response times

## Troubleshooting Ingress Rules

### Common Issues

1. **404 Errors**
   - Verify the service name and port in the Ingress resource
   - Check if the service has endpoints
   - Verify the path and pathType in the Ingress rules

2. **TLS/SSL Issues**
   - Check if the TLS secret exists
   - Verify the secret contains valid certificates
   - Check the Ingress controller logs

3. **Routing Issues**
   - Verify the host and path in the Ingress rules
   - Check if the Ingress controller is running
   - Verify the service is accessible from the Ingress controller

### Diagnostic Commands

```bash
# Check Ingress status
kubectl get ingress

# Describe Ingress for detailed information
kubectl describe ingress <ingress-name>

# Check Ingress controller logs
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Check service endpoints
kubectl get endpoints <service-name>

# Test connectivity from within the cluster
kubectl run -it --rm --image=curlimages/curl curl -- sh
curl http://<service-name>.<namespace>.svc.cluster.local
```

## Next Steps
In the next section, we'll explore IngressClass and how it helps manage different Ingress controller implementations.

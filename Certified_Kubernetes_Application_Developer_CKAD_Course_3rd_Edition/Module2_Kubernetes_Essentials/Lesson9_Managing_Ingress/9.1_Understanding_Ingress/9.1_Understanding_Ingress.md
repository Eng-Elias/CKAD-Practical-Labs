# 9.1 Understanding Ingress

## What is Ingress?

Ingress in Kubernetes is an API object that manages external access to services within a cluster, typically HTTP/HTTPS. It provides:
- External reachable URLs
- Load balancing
- SSL/TLS termination
- Name-based virtual hosting

## Ingress vs Services

| Feature          | Ingress                      | Service (NodePort/LoadBalancer) |
|------------------|------------------------------|--------------------------------|
| Protocol         | HTTP/HTTPS                   | Any TCP/UDP                    |
| Layer            | Application (Layer 7)        | Transport (Layer 4)            |
| Routing          | Path/URL based               | Simple load balancing          |
| SSL Termination  | Built-in                     | Requires additional setup      |
| Implementation   | Requires Ingress Controller  | Built into kube-proxy          |

## How Ingress Works

1. **User Request**: User accesses a URL (e.g., myapp.example.com)
2. **DNS Resolution**: DNS resolves to the Ingress Controller's IP
3. **Ingress Controller**: Receives the request and processes it based on Ingress rules
4. **Routing**: Forwards traffic to the appropriate Service based on the rules
5. **Service**: Routes to the backend Pods

![Ingress Flow](https://kubernetes.io/docs/concepts/services-networking/ingress.svg)

## Ingress Controller

An Ingress Controller is responsible for fulfilling the Ingress, usually with a load balancer. Common implementations include:

- **NGINX Ingress Controller**
- **HAProxy Ingress**
- **Traefik**
- **Contour**
- **AWS ALB Ingress Controller**

## Basic Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

## Key Components

### 1. Rules
Define how traffic should be routed based on the request host and path.

### 2. Path Types
- **Exact**: Matches the URL path exactly
- **Prefix**: Matches based on a URL path prefix
- **ImplementationSpecific**: Interpretation depends on the IngressClass

### 3. Default Backend
Handles requests that don't match any rule.

## Common Annotations

| Annotation | Description |
|------------|-------------|
| `nginx.ingress.kubernetes.io/rewrite-target` | URL rewrite target |
| `kubernetes.io/ingress.class` | Specifies the Ingress controller |
| `cert-manager.io/issuer` | Reference to a cert-manager issuer |
| `nginx.ingress.kubernetes.io/ssl-redirect` | Redirect HTTP to HTTPS |

## Best Practices

1. **Use meaningful names** for Ingress resources
2. **Implement HTTPS** for all production traffic
3. **Use path-based routing** for microservices
4. **Set up monitoring** for your Ingress controller
5. **Configure resource limits** for the Ingress controller

## Common Use Cases

1. **Name-based Virtual Hosting**
   - Multiple domains pointing to different services
   - Example: blog.example.com → blog-service

2. **Path-based Routing**
   - Different paths route to different services
   - Example: example.com/api → api-service

3. **TLS Termination**
   - Offload SSL/TLS at the Ingress controller
   - Centralized certificate management

## Next Steps
In the next section, we'll configure the Minikube Ingress controller to enable Ingress functionality in our local development environment.

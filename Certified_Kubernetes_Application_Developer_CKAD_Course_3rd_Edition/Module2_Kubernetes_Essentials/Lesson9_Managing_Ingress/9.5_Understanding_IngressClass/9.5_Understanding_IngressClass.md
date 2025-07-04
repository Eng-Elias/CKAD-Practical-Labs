# 9.5 Understanding IngressClass

## Introduction to IngressClass

IngressClass is a Kubernetes resource that was introduced in version 1.18 and became stable in 1.22. It represents the class of the Ingress controller that should be used for a particular Ingress resource.

## Why Use IngressClass?

1. **Multiple Ingress Controllers**: Run different Ingress controllers in the same cluster
2. **Custom Configurations**: Apply specific configurations to different Ingress resources
3. **Default Controller**: Set a default Ingress controller for the cluster
4. **Cleaner Configuration**: Move controller-specific configurations out of annotations

## IngressClass Resource

### Basic Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx-example
spec:
  controller: k8s.io/ingress-nginx
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
    namespace: external-configuration
```

### Key Fields

- **controller**: String that references the controller name
- **parameters**: Optional reference to additional configuration

## Default IngressClass

### Marking an IngressClass as Default

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```

### Verifying Default IngressClass

```bash
kubectl get ingressclass
```

## Using IngressClass with Ingress

### Specifying IngressClass in Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /test
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

## Common IngressClass Examples

### 1. NGINX Ingress Controller

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```

### 2. Contour Ingress Controller

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: contour
  annotations:
    ingressclass.kubernetes.io/is-default-class: "false"
spec:
  controller: projectcontour.io/ingress-controller
```

## Migrating from Annotations to IngressClass

### Old Way (Using Annotation)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

### New Way (Using IngressClass)

```yaml
# First, create the IngressClass
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
---
# Then reference it in your Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

## Best Practices

1. **Always Specify IngressClass**
   - Explicitly set the `ingressClassName` field in your Ingress resources
   - Avoid relying on default IngressClass unless absolutely necessary

2. **Naming Conventions**
   - Use descriptive names that indicate the purpose of the IngressClass
   - Include the controller type in the name (e.g., `nginx-external`, `nginx-internal`)

3. **Documentation**
   - Document the purpose of each IngressClass
   - Include any specific requirements or limitations

4. **RBAC**
   - Set up proper RBAC for IngressClass resources
   - Restrict who can create or modify IngressClass resources

## Troubleshooting

### Common Issues

1. **No Matching IngressClass**
   ```
   Error: no IngressClass with name "nginx" found
   ```
   **Solution**: Create the IngressClass or use an existing one

2. **Multiple Default IngressClasses**
   ```
   Warning: More than one default IngressClass found
   ```
   **Solution**: Ensure only one IngressClass is marked as default

3. **Controller Not Found**
   ```
   Error: failed to create IngressClass: admission webhook "validate.nginx.ingress.kubernetes.io" denied the request
   ```
   **Solution**: Verify the controller name is correct and the controller is installed

### Diagnostic Commands

```bash
# List all IngressClasses
kubectl get ingressclass

# Describe a specific IngressClass
kubectl describe ingressclass <name>

# Check Ingress controller logs
kubectl logs -n <namespace> <ingress-controller-pod>

# Check events for Ingress resources
kubectl get events --field-selector involvedObject.kind=Ingress
```

## Migration Guide

### From Annotations to IngressClass

1. **Identify Existing Ingress Resources**
   ```bash
   kubectl get ingress --all-namespaces -o=jsonpath="{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.metadata.annotations.\"kubernetes\.io/ingress\.class\"}{'\n'}{end}"
   ```

2. **Create IngressClass Resources**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: IngressClass
   metadata:
     name: nginx
   spec:
     controller: k8s.io/ingress-nginx
   ```

3. **Update Ingress Resources**
   ```bash
   kubectl patch ingress <ingress-name> -n <namespace> -p '{"spec":{"ingressClassName":"nginx"}}'
   ```

4. **Verify the Migration**
   ```bash
   kubectl get ingress --all-namespaces -o=jsonpath="{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.spec.ingressClassName}{'\n'}{end}"
   ```

## Next Steps
In the next section, we'll cover troubleshooting common Ingress issues and how to debug them effectively.

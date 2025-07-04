# 8.2 Understanding Services

## What are Kubernetes Services?

Kubernetes Services are an abstraction that defines:
- A logical set of Pods
- A policy to access them
- A stable endpoint for applications

### Key Characteristics
- Services provide stable IP addresses and DNS names
- They enable load balancing across Pods
- They decouple frontend and backend workloads

## Service Types

### 1. ClusterIP (Default)
- Creates a virtual IP inside the cluster
- Only accessible within the cluster
- Perfect for internal service-to-service communication

### 2. NodePort
- Exposes the service on each Node's IP at a static port
- Makes a service accessible from outside the cluster
- Automatically creates a ClusterIP service

### 3. LoadBalancer
- Creates an external load balancer in cloud providers
- Automatically creates NodePort and ClusterIP services
- Integrates with cloud provider's load balancer

## Core Components

### kube-proxy
- Runs on each node
- Maintains network rules on nodes
- Handles service IP to pod IP translation

### Endpoints
- Dynamically updated list of healthy Pods
- Managed by the Endpoints controller
- Used by kube-proxy for load balancing

## Creating a Basic Service

### Using kubectl expose
```bash
# Create a deployment
kubectl create deployment nginx --image=nginx

# Expose the deployment as a service
kubectl expose deployment nginx --port=80 --target-port=80

# Verify the service
kubectl get services
```

### Using YAML Manifest
```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

## Service Discovery

### Environment Variables
- Automatically injected into Pods
- Example: `NGINX_SERVICE_SERVICE_HOST`
- Only available to Pods created after the Service

### DNS
- Preferred method for service discovery
- Format: `<service-name>.<namespace>.svc.cluster.local`
- Example: `nginx-service.default.svc.cluster.local`

## Advanced Service Configuration

### Session Affinity
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  sessionAffinity: ClientIP
  # ...
```

### Multiple Ports
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
    - name: https
      protocol: TCP
      port: 443
      targetPort: 8443
```

## Troubleshooting Services

### Common Issues
1. **Service not accessible**
   - Check if Pods are running and have correct labels
   - Verify service selector matches Pod labels
   - Check service endpoints: `kubectl get endpoints <service-name>`

2. **DNS resolution not working**
   - Check CoreDNS pods
   - Verify DNS configuration in kubelet
   - Test DNS resolution from a Pod

3. **Network policies blocking traffic**
   - Check NetworkPolicy resources
   - Verify namespace isolation
   - Check CNI plugin logs

## Best Practices

1. **Use meaningful names**
   - Choose descriptive service names
   - Follow naming conventions
   - Include version information if needed

2. **Resource management**
   - Set appropriate resource requests/limits
   - Monitor service performance
   - Use Horizontal Pod Autoscaler when needed

3. **Security**
   - Use NetworkPolicies to restrict traffic
   - Implement proper authentication/authorization
   - Regularly update and patch

## Next Steps
In the next section, we'll explore how to create and manage different types of services in Kubernetes.

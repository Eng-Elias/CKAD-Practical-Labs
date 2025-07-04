# 4.7 Running Your First Application

## Prerequisites
- Minikube installed and running
- kubectl configured to communicate with your cluster

## Running a Simple Application

### 1. Create a Deployment
```bash
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
```

### 2. Verify the Deployment
```bash
kubectl get deployments
kubectl get pods
```

### 3. Expose the Deployment as a Service
```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

### 4. Access the Application
```bash
minikube service hello-minikube
```
Or get the URL:
```bash
minikube service hello-minikube --url
```

## Understanding kubectl Basics

### Common kubectl Commands
```bash
# Get cluster information
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get all resources
kubectl get all

# Describe a resource
kubectl describe pod/<pod-name>

# View logs
kubectl logs <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/sh
```

## Working with YAML Manifests

### 1. Create a Deployment YAML
```bash
kubectl create deployment nginx --image=nginx:1.14.2 --dry-run=client -o yaml > nginx-deployment.yaml
```

### 2. Edit the YAML (optional)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
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
```

### 3. Apply the YAML
```bash
kubectl apply -f nginx-deployment.yaml
```

## Scaling Applications

### Scale a Deployment
```bash
kubectl scale deployment/nginx --replicas=5
```

### Verify Scaling
```bash
kubectl get pods
kubectl get deployments
```

## Cleaning Up

### Delete Resources
```bash
# Delete by name
kubectl delete deployment hello-minikube
kubectl delete service hello-minikube

# Or delete using the YAML file
kubectl delete -f nginx-deployment.yaml
```

### Stop Minikube
```bash
minikube stop
```

## Common Issues and Solutions

### 1. Pod Stays in Pending State
```bash
# Check events
kubectl get events --sort-by='.metadata.creationTimestamp'

# Describe the pod
kubectl describe pod/<pod-name>
```

### 2. Service Not Accessible
```bash
# Check service details
kubectl describe service <service-name>

# Check endpoints
kubectl get endpoints <service-name>
```

### 3. Image Pull Errors
```bash
# Check pod status
kubectl describe pod/<pod-name>

# Try pulling the image manually
docker pull <image-name>
```

## Best Practices
1. Always use Deployments instead of creating Pods directly
2. Use meaningful names for resources
3. Clean up resources you're not using
4. Use namespaces to organize resources
5. Always check resource usage and logs when debugging

## Next Steps
- Learn about Kubernetes objects and their relationships
- Explore more advanced deployment strategies
- Practice with different types of workloads

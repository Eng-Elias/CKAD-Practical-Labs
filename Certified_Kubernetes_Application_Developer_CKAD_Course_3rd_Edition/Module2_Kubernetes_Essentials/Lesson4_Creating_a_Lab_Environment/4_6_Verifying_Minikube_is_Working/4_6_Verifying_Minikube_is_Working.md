# 4.6 Verifying Minikube is Working

## Basic Verification Commands

### 1. Check Minikube Status
```bash
minikube status
```
Expected output:
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### 2. Check Kubernetes Cluster Info
```bash
kubectl cluster-info
```

### 3. List All Cluster Components
```bash
kubectl get all --all-namespaces
```

## Testing with a Sample Application

### 1. Create a Test Deployment
```bash
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
```

### 2. Expose the Deployment
```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

### 3. Access the Application
```bash
minikube service hello-minikube --url
```
Or open directly in browser:
```bash
minikube service hello-minikube
```

## Advanced Verification

### 1. Check Node Status
```bash
kubectl get nodes -o wide
```

### 2. Check System Pods
```bash
kubectl get pods -n kube-system
```

### 3. Check Cluster Events
```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
```

## Using Minikube Dashboard

### 1. Start the Dashboard
```bash
minikube dashboard
```
This will open the Kubernetes dashboard in your default web browser.

### 2. Access Dashboard via Proxy
```bash
kubectl proxy
```
Then visit: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

## Common Issues and Solutions

### 1. Minikube Fails to Start
```bash
# Check logs
minikube logs

# Delete and recreate
minikube delete
minikube start
```

### 2. Dashboard Not Accessible
```bash
# Enable dashboard addon
minikube addons enable dashboard

# Check service status
kubectl get svc -n kubernetes-dashboard
```

### 3. Insufficient Resources
```bash
# Check resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Allocate more resources
minikube stop
minikube config set memory 4096
minikube config set cpus 2
minikube start
```

## Cleanup
```bash
# Delete test resources
kubectl delete service hello-minikube
kubectl delete deployment hello-minikube

# Reset Minikube (if needed)
minikube stop
minikube delete
minikube start
```

## Next Steps
- Explore Minikube add-ons: `minikube addons list`
- Try deploying a sample application
- Proceed with CKAD course exercises

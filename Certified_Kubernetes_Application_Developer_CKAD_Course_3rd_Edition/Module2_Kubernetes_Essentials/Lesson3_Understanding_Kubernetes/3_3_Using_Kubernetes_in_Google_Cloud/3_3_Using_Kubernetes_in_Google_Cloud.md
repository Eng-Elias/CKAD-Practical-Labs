# 3.3 Using Kubernetes in Google Cloud

## Introduction to Google Kubernetes Engine (GKE)

- GKE is Google Cloud's managed Kubernetes service
- Provides an easy way to get started with Kubernetes
- Handles management of the control plane and nodes
- Ideal for learning and production workloads

## Getting Started with GKE

### Prerequisites
- Google Cloud account (Free tier available with $300 credit)
- Basic understanding of cloud concepts

### Creating Your First Cluster

1. **Access Google Cloud Console**
   - Navigate to Kubernetes Engine
   - Click on "Clusters" in the left sidebar
   - Click "Create"

2. **Cluster Configuration**
   - Select "GKE Standard"
   - Choose "My first cluster" for learning purposes
   - Default settings are sufficient for initial exploration

### Cluster Specifications

- **Nodes**: 3 (default for high availability)
- **Location**: Automatically selected based on your account settings
- **Machine Type**: Optimized for cost-effective learning
- **Estimated Cost**: ~$4/day in US Central region

## Connecting to Your Cluster

1. **Using Cloud Shell**
   - Click "Connect" next to your cluster
   - Select "Run in Cloud Shell"
   - Authenticate when prompted

2. **Verifying the Connection**
   ```bash
   # Check cluster nodes
   kubectl get nodes
   
   # View all resources
   kubectl get all
   ```

## Running Your First Application

```bash
# Create a simple Nginx deployment
kubectl create deployment my-nginx --image=nginx

# Scale the deployment
kubectl scale deployment my-nginx --replicas=3

# Expose the deployment
kubectl expose deployment my-nginx --port=80 --type=LoadBalancer

# Check the service
kubectl get services
```

## Important Considerations

- **Cost Management**
  - Remember to delete resources when not in use
  - Use preemptible nodes for development to reduce costs

- **Best Practices**
  - Use meaningful names for resources
  - Tag resources for better organization
  - Set up monitoring and logging

## Next Steps

- Explore cluster autoscaling
- Set up persistent storage
- Configure networking and security policies
- Implement CI/CD pipelines

## Cleaning Up

```bash
# Delete the service
kubectl delete service my-nginx

# Delete the deployment
kubectl delete deployment my-nginx
```

> **Note:** Always clean up resources to avoid unexpected charges when using cloud services.

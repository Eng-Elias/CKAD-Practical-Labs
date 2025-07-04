# 13.4 Implementing Blue-Green Deployments

## Introduction to Blue-Green Deployments

Blue-Green deployment is a strategy that minimizes downtime and risk by running two identical production environments called Blue and Green. At any time, only one of the environments is live, with the live environment serving all production traffic.

## Key Concepts

### Blue Environment
- Current production environment
- Serves all live traffic
- Represents the stable version of the application

### Green Environment
- New version of the application
- Deployed alongside Blue
- Not receiving production traffic initially
- Used for testing before switching traffic

## Implementation Steps

### 1. Set Up the Blue Environment

```bash
# Create the initial deployment (Blue)
kubectl create deployment blue-nginx --image=nginx:1.14 --replicas=3

# Create a service to expose the deployment
kubectl expose deployment blue-nginx --port=80 --name=nginx-service

# Verify the service is working
kubectl get svc nginx-service
```

### 2. Create the Green Environment

```yaml
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-nginx
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
        image: nginx:1.17  # New version
        ports:
        - containerPort: 80
```

Apply the green deployment:
```bash
kubectl apply -f green-deployment.yaml
```

### 3. Test the Green Environment

```bash
# Create a temporary service for testing
kubectl expose deployment green-nginx --port=80 --name=green-service

# Get the service IP
kubectl get svc green-service

# Test the green service (replace with actual IP)
curl <green-service-ip>
```

### 4. Switch Traffic from Blue to Green

```bash
# Delete the current service
kubectl delete svc nginx-service

# Create a new service pointing to green
kubectl expose deployment green-nginx --port=80 --name=nginx-service
```

### 5. Verify and Clean Up

```bash
# Verify traffic is going to green
kubectl get svc nginx-service
kubectl get endpoints nginx-service

# Once verified, delete the blue deployment
kubectl delete deployment blue-nginx

# Delete the temporary green service
kubectl delete svc green-service
```

## Best Practices

1. **Automated Testing**: Implement automated testing in the green environment before switching traffic
2. **Monitoring**: Monitor both environments during the transition
3. **Rollback Plan**: Be prepared to quickly switch back to blue if issues arise
4. **Database Migrations**: Plan database schema changes carefully to maintain compatibility
5. **Session Persistence**: Consider how user sessions will be handled during the switch

## Common Issues and Solutions

### Issue: Service IP Changes
- **Symptom**: Clients experience connection issues after switch
- **Solution**: Use a service mesh or ingress controller that supports zero-downtime deployments

### Issue: Database Schema Incompatibility
- **Symptom**: New version fails due to database changes
- **Solution**: Implement backward-compatible database migrations

### Issue: Sticky Sessions
- **Symptom**: Users experience session loss during switch
- **Solution**: Use a distributed session store or implement session migration

## Example: Complete Blue-Green Deployment Script

```bash
#!/bin/bash

# Set colors for output
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to deploy a new version
deploy_green() {
    echo -e "${GREEN}Deploying new version (Green)...${NC}"
    kubectl apply -f green-deployment.yaml
    
    echo -e "${GREEN}Creating temporary service for testing...${NC}"
    kubectl expose deployment green-nginx --port=80 --name=green-service
    
    # Wait for pods to be ready
    kubectl rollout status deployment/green-nginx
    
    echo -e "${GREEN}Green deployment is ready for testing${NC}"
}

# Function to switch traffic to green
switch_to_green() {
    echo -e "${GREEN}Switching traffic to Green...${NC}"
    
    # Delete the current service
    kubectl delete svc nginx-service
    
    # Create new service pointing to green
    kubectl expose deployment green-nginx --port=80 --name=nginx-service
    
    echo -e "${GREEN}Traffic switched to Green${NC}"
}

# Function to rollback to blue
rollback_to_blue() {
    echo -e "${BLUE}Rolling back to Blue...${NC}"
    
    # Delete the green service
    kubectl delete svc nginx-service
    
    # Recreate the blue service
    kubectl expose deployment blue-nginx --port=80 --name=nginx-service
    
    echo -e "${BLUE}Rollback to Blue completed${NC}"
}

# Main execution
echo "Starting Blue-Green deployment..."

deploy_green

# Ask for confirmation before switching
read -p "Test the green deployment. Switch traffic to green? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]
then
    switch_to_green
    
    # Ask if we should keep the blue deployment
    read -p "Keep blue deployment for rollback? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Nn]$ ]]
    then
        kubectl delete deployment blue-nginx
    fi
else
    echo "Keeping blue deployment"
    kubectl delete deployment green-nginx
    kubectl delete svc green-service
fi

echo "Blue-Green deployment process completed"

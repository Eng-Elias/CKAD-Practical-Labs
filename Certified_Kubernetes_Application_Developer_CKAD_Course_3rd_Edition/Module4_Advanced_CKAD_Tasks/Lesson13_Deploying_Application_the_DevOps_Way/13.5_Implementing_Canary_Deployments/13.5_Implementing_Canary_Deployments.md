# 13.5 Implementing Canary Deployments

## Introduction to Canary Deployments

Canary deployment is a technique to reduce the risk of introducing a new software version in production by gradually rolling out the change to a small subset of users before rolling it out to the entire infrastructure and making it available to everybody.

## Key Concepts

### Canary Deployment Characteristics
- Gradual rollout of new features
- Real-user testing in production
- Quick rollback if issues are detected
- Minimized impact of potential failures

## Implementation Approaches

### 1. Using Replica Counts (Simple Canary)

```bash
# Initial deployment (90% of traffic)
kubectl create deployment myapp --image=myapp:v1 --replicas=9

# Canary deployment (10% of traffic)
kubectl create deployment myapp-canary --image=myapp:v2 --replicas=1

# Create a service that selects both deployments
kubectl expose deployment myapp --port=80 --name=myapp-service
```

### 2. Using Service Mesh (Advanced Canary)

Using Istio for traffic splitting:

```yaml
# virtual-service.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - "*"
  http:
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## Step-by-Step Implementation

### 1. Create the Base Deployment

```yaml
# deployment-base.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-base
  labels:
    app: myapp
    track: stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: myapp
        image: myapp:v1
        ports:
        - containerPort: 80
```

### 2. Create the Canary Deployment

```yaml
# deployment-canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
  labels:
    app: myapp
    track: canary
spec:
  replicas: 1  # Start with 25% of traffic (1/4)
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2
        ports:
        - containerPort: 80
```

### 3. Create a Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp  # Matches both base and canary
```

### 4. Apply the Configuration

```bash
# Apply configurations
kubectl apply -f deployment-base.yaml
kubectl apply -f deployment-canary.yaml
kubectl apply -f service.yaml

# Verify the deployment
kubectl get pods -l app=myapp
```

## Monitoring and Scaling

### Monitor the Canary

```bash
# Watch the pods
kubectl get pods -l app=myapp -w

# Check the logs
kubectl logs -l app=myapp,track=canary

# Monitor metrics (if available)
kubectl top pod -l app=myapp
```

### Scale the Canary

```bash
# After verifying the canary, scale up
kubectl scale deployment myapp-canary --replicas=2  # 40% traffic

# Eventually, replace the base deployment
kubectl scale deployment myapp-base --replicas=0
kubectl scale deployment myapp-canary --replicas=3  # 100% traffic

# Update the base deployment image for future updates
kubectl set image deployment/myapp-base myapp=myapp:v2
kubectl scale deployment myapp-base --replicas=3
kubectl scale deployment myapp-canary --replicas=0
```

## Best Practices

1. **Start Small**: Begin with a small percentage of traffic (5-10%)
2. **Monitor Closely**: Watch metrics and logs for any issues
3. **Automate Rollback**: Have automated rollback procedures in place
4. **Test in Staging**: Test the canary in a staging environment first
5. **Use Labels**: Clearly label canary deployments for easy identification

## Advanced Canary with Prometheus and Grafana

### Monitoring Setup

```yaml
# prometheus-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        action: keep
        regex: myapp
```

### Alert Rules

```yaml
# prometheus-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-rules
  labels:
    app: myapp
spec:
  groups:
  - name: myapp.rules
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate on {{ $labels.instance }}"
        description: "{{ $value }}% of requests are failing"
```

## Rollback Procedure

```bash
# If issues are detected, rollback immediately
kubectl scale deployment myapp-canary --replicas=0

# Or delete the canary deployment
kubectl delete deployment myapp-canary
```

## Complete Canary Deployment Script

```bash
#!/bin/bash

# Configuration
APP_NAME="myapp"
VERSION="v2"
NAMESPACE="default"
TOTAL_REPLICAS=4  # 3 stable + 1 canary = 4 total (25% canary)

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Function to deploy canary
deploy_canary() {
    echo -e "${YELLOW}Deploying canary version ${VERSION}...${NC}"
    
    # Create canary deployment
    cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${APP_NAME}-canary
  labels:
    app: ${APP_NAME}
    track: canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${APP_NAME}
  template:
    metadata:
      labels:
        app: ${APP_NAME}
        track: canary
        version: ${VERSION}
    spec:
      containers:
      - name: ${APP_NAME}
        image: ${APP_NAME}:${VERSION}
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
EOF
    
    echo -e "${YELLOW}Waiting for canary to be ready...${NC}"
    kubectl rollout status deployment/${APP_NAME}-canary --timeout=300s
    
    if [ $? -eq 0 ]; then
        echo -e "${GREEN}Canary deployment successful!${NC}"
        return 0
    else
        echo -e "${RED}Canary deployment failed!${NC}"
        return 1
    fi
}

# Function to promote canary
promote_canary() {
    echo -e "${YELLOW}Promoting canary to production...${NC}"
    
    # Scale down old version
    kubectl scale deployment ${APP_NAME} --replicas=0
    
    # Scale up canary
    kubectl scale deployment ${APP_NAME}-canary --replicas=${TOTAL_REPLICAS}
    
    # Update the main deployment to use new image
    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${APP_NAME}:${VERSION}
    
    echo -e "${GREEN}Promotion complete!${NC}"
}

# Function to rollback canary
rollback_canary() {
    echo -e "${RED}Rolling back canary...${NC}"
    
    # Scale down canary
    kubectl scale deployment ${APP_NAME}-canary --replicas=0
    
    # Ensure main deployment is running
    kubectl scale deployment ${APP_NAME} --replicas=${TOTAL_REPLICAS}
    
    echo -e "${GREEN}Rollback complete!${NC}"
}

# Main execution
echo "Starting canary deployment for ${APP_NAME} ${VERSION}..."

deploy_canary

if [ $? -eq 0 ]; then
    read -p "Canary deployed successfully. Promote to production? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        promote_canary
    else
        read -p "Rollback canary? (y/n) " -n 1 -r
        echo
        if [[ $REPLY =~ ^[Yy]$ ]]; then
            rollback_canary
        fi
    fi
else
    echo -e "${RED}Canary deployment failed. Initiating rollback...${NC}"
    rollback_canary
    exit 1
fi

echo "Canary deployment process completed"

# 6.7 Managing Resource Limitations and Quota

## Introduction to Resource Management

Resource management in Kubernetes ensures that applications get the compute resources they need while preventing resource contention. This is crucial for maintaining cluster stability and performance.

### Key Concepts
- **Requests**: Minimum amount of resources guaranteed to a container
- **Limits**: Maximum amount of resources a container can use
- **Quotas**: Resource limits at the namespace level
- **Limit Ranges**: Default and limit constraints for resources in a namespace

## Resource Requests and Limits

### Basic Pod with Resource Requirements
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: nginx:1.19
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Understanding Resource Units
- **CPU**: 1 CPU = 1000m (millicores)
  - `500m` = 0.5 CPU
  - `1000m` = 1 CPU
- **Memory**: Can be specified in bytes, or with SI suffixes (K, M, G, T, P, E)
  - `64M` = 64 Mebibytes
  - `1Gi` = 1 Gibibyte (1024^3 bytes)

## Resource Quotas

### Creating a Resource Quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-demo
  namespace: default
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "8"
    limits.memory: 8Gi
    requests.storage: 50Gi
    persistentvolumeclaims: "5"
    services.nodeports: "5"
```

### Viewing Quota Usage
```bash
# List all resource quotas
kubectl get resourcequota --all-namespaces

# Describe a specific quota
kubectl describe resourcequota pod-demo -n default

# View quota usage in a namespace
kubectl get resourcequota --namespace=default
```

## Limit Ranges

### Creating a Limit Range
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limit-range
  namespace: default
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
    type: Container
```

### Viewing Limit Ranges
```bash
# List limit ranges in current namespace
kubectl get limitrange

# View details of a limit range
kubectl describe limitrange default-limit-range
```

## Quality of Service (QoS) Classes

Kubernetes assigns QoS classes to Pods based on their resource requests and limits:

### 1. Guaranteed
- All containers have memory and CPU limits and requests set
- Limits equal requests for all containers
- Highest priority for scheduling and eviction

```yaml
resources:
  limits:
    memory: "128Mi"
    cpu: "500m"
  requests:
    memory: "128Mi"
    cpu: "500m"
```

### 2. Burstable
- At least one container has a memory or CPU request
- Pod doesn't meet the criteria for Guaranteed QoS
- Medium priority for scheduling and eviction

```yaml
resources:
  limits:
    memory: "256Mi"
  requests:
    memory: "128Mi"
```

### 3. BestEffort
- No memory or CPU limits or requests set
- Lowest priority for scheduling and eviction
- First to be terminated under resource pressure

```yaml
# No resources section
```

## Monitoring Resource Usage

### Using kubectl top
```bash
# Show metrics for nodes
kubectl top node

# Show metrics for pods
kubectl top pod

# Show metrics for pods in all namespaces
kubectl top pod --all-namespaces

# Show metrics for containers in a pod
kubectl top pod my-pod --containers
```

### Using Metrics Server
```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify installation
kubectl get deployment metrics-server -n kube-system
```

## Practical Examples

### Multi-container Pod with Resources
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: web
    image: nginx:1.19
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
    ports:
    - containerPort: 80
  - name: log-processor
    image: busybox:1.28
    resources:
      requests:
        cpu: "50m"
        memory: "64Mi"
      limits:
        cpu: "100m"
        memory: "128Mi"
    command: ["sh", "-c", "while true; do echo 'Processing logs...'; sleep 10; done"]
```

### Namespace with Resource Quota
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    pods: "20"
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "16"
    limits.memory: 32Gi
    persistentvolumeclaims: "10"
    services.loadbalancers: "3"
    services.nodeports: "5"
```

## Troubleshooting

### Common Issues

#### Pod Pending - Insufficient CPU/Memory
```bash
# Check pod status
kubectl describe pod my-pod

# Look for events like:
# "Insufficient cpu"
# "Insufficient memory"
```

#### Pod Evicted - Exceeded Memory
```bash
# Check pod status
kubectl get pod my-pod -o yaml | grep -A 10 "status:"

# Look for:
# reason: Evicted
# message: 'Pod The node was low on resource: memory.'
```

### Debugging Commands

#### Check Node Capacity
```bash
kubectl describe nodes | grep -A 10 "Allocated resources"
```

#### Check Pod Resource Usage
```bash
kubectl top pod --containers
```

#### Check Quota Usage
```bash
kubectl describe quota --namespace=my-namespace
```

## Best Practices

1. **Set Requests and Limits**
   - Always set resource requests and limits for all containers
   - Start with conservative values and monitor

2. **Right-size Your Containers**
   - Avoid over-provisioning resources
   - Use horizontal pod autoscaling for variable workloads

3. **Use Namespace Quotas**
   - Implement resource quotas per namespace
   - Prevent resource exhaustion from misbehaving applications

4. **Monitor and Adjust**
   - Regularly monitor resource usage
   - Adjust requests and limits based on actual usage patterns

5. **Consider Workload Types**
   - Different applications have different resource profiles
   - Batch jobs vs. web servers have different requirements

## Next Steps
- Learn about Horizontal Pod Autoscaling (HPA)
- Explore Vertical Pod Autoscaling (VPA)
- Understand Cluster Autoscaling
- Study Pod Disruption Budgets for high availability

# 7.8 Understanding AutoScaling

## Introduction to Kubernetes AutoScaling

Kubernetes provides several mechanisms to automatically scale your applications based on demand:

1. **Horizontal Pod Autoscaler (HPA)**: Scales the number of pod replicas
2. **Vertical Pod Autoscaler (VPA)**: Adjusts container CPU and memory requests/limits
3. **Cluster Autoscaler**: Adjusts the size of the node pool

This section focuses on Horizontal Pod Autoscaler (HPA), the most commonly used autoscaling mechanism.

## Horizontal Pod Autoscaler (HPA)

### How HPA Works

1. Monitors resource utilization of pods
2. Compares metrics against target values
3. Adjusts the number of replicas to meet the target
4. Operates within defined minimum and maximum bounds

### Prerequisites for HPA

- Metrics Server must be installed in the cluster
- Resource requests must be set for containers
- The target resource (Deployment, StatefulSet, etc.) must have a defined `.spec.selector`

## Creating an HPA

### Basic Example

First, let's create a deployment with resource requests:

```yaml
# php-apache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

Now, create the HPA:

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Applying the Configuration

```bash
# Create the deployment and service
kubectl apply -f php-apache.yaml

# Create the HPA
kubectl apply -f hpa.yaml

# Check the HPA status
kubectl get hpa
```

## Testing the HPA

### Generating Load

In a separate terminal, run a container to generate load:

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

### Monitoring the HPA

In another terminal, watch the HPA and deployment:

```bash
# Watch HPA
kubectl get hpa -w

# Watch pods
kubectl get pods -w
```

## Advanced HPA Configuration

### Multiple Metrics

HPA can scale based on multiple metrics:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 100Mi
```

### Custom Metrics

HPA can also use custom metrics:

```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests
    target:
      type: AverageValue
      averageValue: 10
```

## Best Practices

1. **Set Appropriate Resource Requests**: HPA needs these to calculate utilization
2. **Configure Reasonable Bounds**: Set appropriate min and max replicas
3. **Monitor HPA Behavior**: Keep an eye on scaling events
4. **Test Under Load**: Validate scaling behavior before production
5. **Consider Stabilization Windows**: For metrics that fluctuate rapidly

## Troubleshooting

### Common Issues

#### HPA Shows "unknown" for metrics

**Possible Causes**:
- Metrics Server not installed or not running
- Resource metrics not available
- Insufficient permissions

**Troubleshooting Steps**:
1. Check if Metrics Server is running:
   ```bash
   kubectl get apiservices | grep metrics
   kubectl get --raw /apis/metrics.k8s.io/
   ```
2. Check Metrics Server logs:
   ```bash
   kubectl logs -n kube-system -l k8s-app=metrics-server
   ```

#### HPA Not Scaling Up/Down

**Possible Causes**:
- Metrics not reaching the target threshold
- Current replicas already at min/max
- Scaling policies preventing changes

**Troubleshooting Steps**:
1. Describe the HPA:
   ```bash
   kubectl describe hpa <hpa-name>
   ```
2. Check metrics directly:
   ```bash
   kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace>/pods
   ```

## Vertical Pod Autoscaler (VPA) - Brief Overview

VPA automatically adjusts the CPU and memory requests and limits of containers:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-app
  updatePolicy:
    updateMode: "Auto"
```

## Cluster Autoscaler - Brief Overview

Cluster Autoscaler automatically adjusts the size of the node pool when:
- Pods fail to run due to insufficient resources
- Nodes are underutilized for an extended period

## Next Steps

You've now learned how to automatically scale your applications in Kubernetes. Consider exploring:

1. Advanced HPA metrics and custom metrics
2. VPA for automatic resource tuning
3. Cluster Autoscaler for node-level scaling
4. KEDA (Kubernetes Event-Driven Autoscaling) for event-driven scaling

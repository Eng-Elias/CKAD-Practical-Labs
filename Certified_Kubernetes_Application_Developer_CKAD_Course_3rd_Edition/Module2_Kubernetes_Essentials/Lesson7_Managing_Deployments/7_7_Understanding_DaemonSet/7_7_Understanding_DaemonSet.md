# 7.7 Understanding DaemonSet

## Introduction to DaemonSets

A DaemonSet ensures that all (or some) nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

## When to Use DaemonSets

- **Node-Level Operations**:
  - Log collection (e.g., Fluentd, Filebeat)
  - Monitoring (e.g., Prometheus Node Exporter)
  - Storage (e.g., GlusterFS, Ceph)
  - Networking (e.g., kube-proxy, CNI plugins)

- **Key Characteristics**:
  - One Pod per node
  - Automatic scheduling on new nodes
  - Node affinity and tolerations support
  - Rolling updates and rollbacks

## Creating a DaemonSet

### Basic Example

```yaml
# nginx-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      name: nginx-daemonset
  template:
    metadata:
      labels:
        name: nginx-daemonset
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            memory: 100Mi
            cpu: 50m
```

### Applying the DaemonSet

```bash
kubectl apply -f nginx-daemonset.yaml

# Verify the DaemonSet
kubectl get daemonsets
kubectl get pods -o wide
```

## Advanced DaemonSet Features

### Node Selection

#### Using nodeSelector

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
```

#### Using Node Affinity

```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
```

### Taints and Tolerations

DaemonSets can be configured to run on tainted nodes:

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "special"
        operator: "Exists"
        effect: "NoSchedule"
```

### Updating a DaemonSet

DaemonSets support rolling updates and rollbacks similar to Deployments:

```bash
# Update the container image
kubectl set image daemonset/nginx-daemonset nginx=nginx:1.20.0

# View the rollout status
kubectl rollout status daemonset/nginx-daemonset

# View the rollout history
kubectl rollout history daemonset/nginx-daemonset

# Rollback to the previous version
kubectl rollout undo daemonset/nginx-daemonset
```

## Practical Example: Node Log Collector

Let's create a DaemonSet that runs a log collector on each node:

```yaml
# log-collector-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  labels:
    app: log-collector
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - name: log-collector
        image: fluent/fluentd:v1.14-1
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Managing DaemonSets

### Common Operations

```bash
# List all DaemonSets
kubectl get daemonsets

# Describe a DaemonSet
kubectl describe daemonset <daemonset-name>

# View DaemonSet events
kubectl describe daemonset <daemonset-name> | grep -A 10 Events:

# Delete a DaemonSet
kubectl delete daemonset <daemonset-name>
```

### Scaling DaemonSets

DaemonSets don't have a `replicas` field since they run one pod per node. However, you can control which nodes they run on using:

1. Node selectors
2. Node affinity/anti-affinity
3. Taints and tolerations

## Best Practices

1. **Resource Limits**: Always set resource requests and limits
2. **Update Strategy**: Choose between `OnDelete` and `RollingUpdate`
3. **Node Selection**: Use node selectors or affinity for specific nodes
4. **Tolerations**: Add tolerations for tainted nodes when necessary
5. **Monitoring**: Monitor DaemonSet pods like any other workload

## Troubleshooting

### Common Issues

#### Issue: DaemonSet Pods Not Creating

**Possible Causes**:
- No nodes match the node selector
- Node has taints without corresponding tolerations
- Resource constraints

**Troubleshooting Steps**:
1. Check DaemonSet status:
   ```bash
   kubectl describe daemonset <daemonset-name>
   ```
2. Check node labels and taints:
   ```bash
   kubectl describe node <node-name>
   ```
3. Check events:
   ```bash
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

#### Issue: Pods in CrashLoopBackOff

**Troubleshooting Steps**:
1. Check pod logs:
   ```bash
   kubectl logs <pod-name>
   ```
2. Describe the pod:
   ```bash
   kubectl describe pod <pod-name>
   ```
3. Check container exit codes:
   ```bash
   kubectl get pod <pod-name> -o jsonpath="{.status.containerStatuses[].lastState.terminated.exitCode}"
   ```

## Next Steps

In the next section, we'll explore Horizontal Pod Autoscaler (HPA) for automatic scaling of your applications.

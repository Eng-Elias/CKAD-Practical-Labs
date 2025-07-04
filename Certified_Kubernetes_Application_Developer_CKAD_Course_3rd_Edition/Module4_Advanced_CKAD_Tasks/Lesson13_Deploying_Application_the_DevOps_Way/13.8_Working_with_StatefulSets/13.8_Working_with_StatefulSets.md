# 13.8 Working with StatefulSets

## Introduction to StatefulSets

StatefulSets are a Kubernetes workload API object that manages the deployment and scaling of a set of Pods, providing guarantees about the ordering and uniqueness of these Pods. Unlike Deployments, StatefulSets maintain a sticky identity for each of their Pods.

## When to Use StatefulSets

- When your application requires stable, unique network identifiers
- When stable, persistent storage is required
- For stateful applications like databases, message brokers, or any application that maintains state
- When ordered, graceful deployment and scaling are required
- When ordered, automated rolling updates are needed

## Key Features of StatefulSets

### Stable Network Identity
- Each pod gets a persistent identifier
- Pods are created in order (0 to N-1)
- Pods are terminated in reverse order (N-1 to 0)

### Stable Storage
- Each pod gets its own PersistentVolume
- Volumes are not deleted when pods are rescheduled
- Volumes are tied to the pod's identity

### Ordered Operations
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

## StatefulSet vs Deployment

| Feature                | StatefulSet                      | Deployment                      |
|------------------------|----------------------------------|---------------------------------|
| Pod Identity          | Stable, unique hostnames         | No guaranteed identity          |
| Pod Naming            | Ordered (web-0, web-1, etc.)     | Random hash suffix              |
| Storage               | Persistent, per-pod              | Ephemeral or shared             |
| Scaling               | Ordered                          | Unordered                       |
| Updates               | Ordered or partitioned           | Rolling or Recreate             |
| Use Case              | Stateful applications            | Stateless applications          |
| Network               | Stable network ID                | Dynamic network ID              |
| DNS                   | Stable DNS hostname              | Dynamic DNS                     |

## Creating a StatefulSet

### Basic StatefulSet Example

```yaml
# nginx-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### Creating the StatefulSet

```bash
# Create the StatefulSet
kubectl apply -f nginx-statefulset.yaml

# List StatefulSets
kubectl get statefulsets
kubectl get pods -l app=nginx

# View the created PVCs
kubectl get pvc

# View the created PVs
kubectl get pv

# View the created service
kubectl get svc nginx
```

## Understanding StatefulSet Pods

### Pod Naming Convention
- StatefulSet pods have a predictable name: `$(statefulset-name)-$(ordinal)`
- For example: `web-0`, `web-1`, `web-2`

### Pod DNS
- Each pod gets a DNS entry: `$(pod-name).$(service-name).$(namespace).svc.cluster.local`
- For example: `web-0.nginx.default.svc.cluster.local`
- Headless service is used for direct pod access

## Scaling StatefulSets

### Scaling Up

```bash
# Scale up to 5 replicas
kubectl scale statefulset web --replicas=5

# Watch the pods being created in order
kubectl get pods -l app=nginx -w
```

### Scaling Down

```bash
# Scale down to 2 replicas
kubectl scale statefulset web --replicas=2

# Verify the pods are terminated in reverse order
kubectl get pods -l app=nginx -w
```

## Updating StatefulSets

### Rolling Update Strategy

```yaml
# nginx-statefulset-update.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  # ... other fields ...
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # Only update pods with ordinal >= 2
  template:
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:1.19
```

### Applying the Update

```bash
# Apply the update
kubectl apply -f nginx-statefulset-update.yaml

# Watch the update progress
kubectl rollout status statefulset web

# List pods to see the updated image
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'
```

## Persistent Storage with StatefulSets

### Volume Claim Templates

```yaml
volumeClaimTemplates:
- metadata:
    name: www
  spec:
    accessModes: [ "ReadWriteOnce" ]
    storageClassName: "standard"
    resources:
      requests:
        storage: 1Gi
```

### Volume Binding Modes
- `WaitForFirstConsumer`: Delay binding and provisioning of a PV until the pod using the PVC is scheduled
- `Immediate`: Bind and provision PVs immediately

### Example with WaitForFirstConsumer

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

## Pod Management Policies

### OrderedReady (Default)
- Pods are created in order (0 to N-1)
- Pods are terminated in reverse order (N-1 to 0)
- If a pod is unhealthy, the StatefulSet will not proceed

### Parallel
- Pods are created and deleted in parallel
- No ordering guarantees
- Faster scaling operations

### Example with Parallel Policy

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  podManagementPolicy: Parallel  # Default is OrderedReady
  replicas: 3
  # ... rest of the spec ...
```

## StatefulSet with Headless Service

### Headless Service Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None  # Makes it a headless service
  selector:
    app: nginx
```

### Benefits of Headless Service
- Direct pod-to-pod communication
- Each pod gets its own DNS entry
- No load balancing or proxying
- Essential for stateful applications that need peer discovery

## StatefulSet with Pod Affinity/Anti-affinity

### Example with Pod Anti-affinity

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Disruption Budget

### Example with PDB

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Init Containers

### Example with Init Container

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      initContainers:
      - name: init
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello from the init container" > /work-dir/index.html']
        volumeMounts:
        - name: workdir
          mountPath: /work-dir
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: workdir
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Topology Spread Constraints

### Example with Topology Spread Constraints

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: nginx
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Readiness Gates

### Example with Readiness Gates

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      readinessGates:
      - conditionType: "www.example.com/feature-1"
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Priority and Preemption

### Example with PriorityClass

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for stateful service pods only."
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      priorityClassName: high-priority
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Security Context

### Example with Security Context

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Resource Limits

### Example with Resource Limits

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Liveness and Readiness Probes

### Example with Probes

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /readiness
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Disruption Budget

### Example with PDB

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Anti-affinity

### Example with Anti-affinity

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet with Pod Topology Spread Constraints

### Example with Topology Spread Constraints

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: nginx
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Best Practices for StatefulSets

### 1. Use Headless Services
- Essential for stable network identity
- Enables direct pod-to-pod communication
- Each pod gets its own DNS entry

### 2. Properly Size Persistent Volumes
- Choose appropriate storage class
- Set appropriate storage size
- Consider performance requirements

### 3. Configure Appropriate Update Strategy
- Use `RollingUpdate` for zero-downtime updates
- Consider using `partition` for canary deployments
- Set appropriate `maxUnavailable` and `maxSurge` values

### 4. Implement Proper Probes
- Configure liveness and readiness probes
- Set appropriate timeouts and thresholds
- Test failure scenarios

### 5. Use Pod Disruption Budgets
- Ensure high availability during maintenance
- Set appropriate `minAvailable` or `maxUnavailable` values
- Test with `kubectl drain`

### 6. Implement Proper Backup and Restore
- Back up persistent volumes
- Test restore procedures
- Consider using Velero or similar tools

### 7. Monitor Stateful Applications
- Monitor pod health
- Monitor storage usage
- Set up alerts for critical conditions

## Common Issues and Solutions

### Issue: Pod Stuck in Terminating State
```
kubectl delete pod web-0 --grace-period=0 --force
```

### Issue: PVC Not Being Deleted
```
kubectl patch pvc www-web-0 -p '{"metadata":{"finalizers":null}}'
```

### Issue: StatefulSet Update Not Working
```
kubectl rollout status statefulset web
kubectl describe statefulset web
kubectl get events --sort-by='.metadata.creationTimestamp'
```

## Conclusion

StatefulSets are a powerful Kubernetes resource for managing stateful applications. They provide stable network identities, persistent storage, and ordered deployment/scaling. By following best practices and understanding their behavior, you can effectively run stateful workloads in Kubernetes.

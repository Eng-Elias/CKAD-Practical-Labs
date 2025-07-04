# 14.4 Monitoring Cluster Event Logs

## Understanding Kubernetes Events

Kubernetes events are objects that provide insight into what is happening inside the cluster. They record important lifecycle events, errors, and state changes for resources like pods, nodes, and services.

### Types of Events

1. **Normal Events**
   - Routine operations
   - Successful scaling
   - Successful scheduling
   - Container creation/destruction

2. **Warning Events**
   - Failed scheduling
   - Image pull errors
   - Container crashes
   - Resource constraints
   - Health check failures

## Viewing Cluster Events

### Basic Event Commands

```bash
# List all events in the current namespace
kubectl get events

# List all events in all namespaces
kubectl get events --all-namespaces

# Watch events in real-time
kubectl get events --watch

# List events sorted by timestamp (newest first)
kubectl get events --sort-by='.metadata.creationTimestamp'

# Filter events by type (Normal/Warning)
kubectl get events --field-selector type=Warning
kubectl get events --field-selector type=Normal
```

### Detailed Event Information

```bash
# Get detailed information about events
kubectl describe events

# Get events for a specific resource
kubectl describe <resource-type>/<resource-name>

# Example: Get events for a specific pod
kubectl describe pod/<pod-name>

# Filter events by reason
kubectl get events --field-selector reason=FailedScheduling

# Get events from a specific namespace
kubectl get events -n <namespace>
```

## Advanced Event Filtering

### Using JSONPath for Custom Filtering

```bash
# Get events where the message contains a specific string
kubectl get events --field-selector involvedObject.kind=Pod -o jsonpath='{range .items[?(@.message=~".*error.*")]}{.message}{"\n"}'

# Get events for a specific pod by name
kubectl get events --field-selector involvedObject.name=<pod-name>

# Get events for a specific node
kubectl get events --field-selector involvedObject.kind=Node,involvedObject.name=<node-name>
```

### Time-based Filtering

```bash
# Get events from the last 5 minutes
kubectl get events --field-selector lastTimestamp>$(date -d '5 minutes ago' -Ins --utc | sed 's/+0000/Z/')

# Get events within a specific time range
kubectl get events --field-selector \
  lastTimestamp>$(date -d '2023-01-01T00:00:00Z' -Ins --utc | sed 's/+0000/Z/'),\
  lastTimestamp<$(date -d '2023-01-02T00:00:00Z' -Ins --utc | sed 's/+0000/Z/')
```

## Common Event Patterns and Solutions

### 1. FailedScheduling

**Symptoms:**
- Pods stuck in Pending state
- Events show "FailedScheduling"

**Troubleshooting:**
```bash
# Get details about the scheduling failure
kubectl get events --field-selector reason=FailedScheduling

# Check node resources
kubectl describe nodes | grep -A 10 "Allocated resources"

# Check node conditions
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check for taints and tolerations
kubectl describe node <node-name> | grep -i taint

# Check pod resource requests
kubectl get pod <pod-name> -o json | jq '.spec.containers[].resources'
```

### 2. ImagePullBackOff

**Symptoms:**
- Pods stuck in ImagePullBackOff state
- Events show image pull errors

**Troubleshooting:**
```bash
# Get details about the image pull failure
kubectl describe pod <pod-name>

# Check the image name and tag
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].image}'

# Try pulling the image manually on a node
docker pull <image-name:tag>

# Check for image pull secrets
kubectl get secret --all-namespaces | grep -i secret
kubectl describe pod <pod-name> | grep -i imagepull
```

### 3. CrashLoopBackOff

**Symptoms:**
- Pods restarting repeatedly
- CrashLoopBackOff in pod status

**Troubleshooting:**
```bash
# Get the reason for the crash
kubectl logs <pod-name> --previous

# Check container exit code
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[].state.waiting.reason}'

# Check resource limits
kubectl describe pod <pod-name> | grep -A 5 "Limits"

# Check application logs
kubectl logs <pod-name>
```

## Setting Up Event Monitoring

### Event Exporter

Deploy an event exporter to collect and analyze Kubernetes events:

```yaml
# event-exporter.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: event-exporter
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: event-exporter
  template:
    metadata:
      labels:
        app: event-exporter
    spec:
      containers:
      - name: event-exporter
        image: ghcr.io/resmoio/kubernetes-event-exporter:latest
        args:
          - --config=/etc/event-exporter/config.yaml
        volumeMounts:
        - name: config
          mountPath: /etc/event-exporter
      volumes:
      - name: config
        configMap:
          name: event-exporter-cfg
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: event-exporter-cfg
  namespace: kube-system
data:
  config.yaml: |
    logLevel: info
    logFormat: json
    route:
      routes:
        - match:
            - receiver: console
    receivers:
      - name: console
        stdout: {}
```

### Prometheus and Grafana Setup

1. Install kube-prometheus-stack:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack
   ```

2. Access Grafana:
   ```bash
   kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
   # Open http://localhost:3000
   # Default credentials: admin/prom-operator
   ```

3. Import Kubernetes Cluster Monitoring dashboard (ID: 315)

## Log Aggregation

### EFK Stack (Elasticsearch, Fluentd, Kibana)

1. Install Elasticsearch:
   ```bash
   kubectl apply -f https://download.elastic.co/downloads/eck/2.9.0/crds.yaml
   kubectl apply -f https://download.elastic.co/downloads/eck/2.9.0/operator.yaml
   
   # Create Elasticsearch cluster
   cat <<EOF | kubectl apply -f -
   apiVersion: elasticsearch.k8s.elastic.co/v1
   kind: Elasticsearch
   metadata:
     name: quickstart
   spec:
     version: 8.9.0
     nodeSets:
     - name: default
       count: 1
       config:
         node.store.allow_mmap: false
   EOF
   ```

2. Install Kibana:
   ```bash
   cat <<EOF | kubectl apply -f -
   apiVersion: kibana.k8s.elastic.co/v1
   kind: Kibana
   metadata:
     name: quickstart
   spec:
     version: 8.9.0
     count: 1
     elasticsearchRef:
       name: quickstart
   EOF
   ```

3. Install Fluentd:
   ```bash
   kubectl create namespace logging
   kubectl create -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml
   ```

## Custom Event Monitoring

### Creating a Custom Event Watcher

```python
from kubernetes import client, config, watch

def watch_events():
    # Load kubeconfig
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    # Create a watch object
    w = watch.Watch()
    
    # Watch for events
    for event in w.stream(v1.list_event_for_all_namespaces):
        print(f"Event: {event['type']} {event['object'].metadata.name}")
        print(f"Reason: {event['object'].reason}")
        print(f"Message: {event['object'].message}")
        print("-" * 50)

if __name__ == "__main__":
    watch_events()
```

## Best Practices for Event Monitoring

1. **Centralized Logging**
   - Aggregate logs from all cluster components
   - Store logs with appropriate retention policies
   - Implement log rotation

2. **Alerting**
   - Set up alerts for critical events
   - Use tools like Alertmanager with Prometheus
   - Configure notification channels (Slack, Email, PagerDuty)

3. **Retention and Archiving**
   - Define retention policies for events
   - Archive important events for compliance
   - Consider using external storage for long-term retention

4. **Performance Considerations**
   - Be mindful of the volume of events
   - Use filters to reduce noise
   - Consider event sampling in high-volume clusters

5. **Security**
   - Restrict access to event data
   - Implement RBAC for event access
   - Audit event access and modifications

## Troubleshooting Common Event Issues

### Missing Events

```bash
# Check if the API server is running
kubectl get --raw /api/v1/namespaces/default/events

# Check API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check etcd health
kubectl get --raw /healthz/etcd
```

### Event Backpressure

```bash
# Check API server metrics
kubectl get --raw /metrics | grep apiserver_storage_events

# Check etcd metrics
kubectl get --raw /metrics | grep etcd_mvcc_db_total_size_in_bytes

# Check API server storage size
kubectl get --raw /metrics | grep apiserver_storage_size_bytes
```

### Event Duplication

```bash
# Check for duplicate events
kubectl get events --field-selector type=Warning --no-headers | \
  awk '{print $2,$3,$4,$5,$6,$7}' | sort | uniq -c | sort -nr | head

# Check event compaction settings
kubectl get cm -n kube-system kube-apiserver -o yaml | grep -A 10 event-ttl
```

## Conclusion

Effective monitoring of cluster event logs is crucial for maintaining a healthy Kubernetes environment. By understanding how to access, filter, and analyze events, you can quickly identify and resolve issues. Implementing a robust monitoring solution with proper alerting and log aggregation will help you stay ahead of potential problems and maintain optimal cluster performance.

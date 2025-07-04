# 14.6 Using Probes

## Understanding Kubernetes Probes

Kubernetes provides three types of probes to monitor container health:

1. **Liveness Probes** - Determines when to restart a container
2. **Readiness Probes** - Determines when a container is ready to serve traffic
3. **Startup Probes** - Used when an application needs a longer time to start up

## Probe Configuration Options

### Common Probe Parameters

```yaml
livenessProbe:  # or readinessProbe/startupProbe
  # How to perform the probe
  httpGet:        # HTTP endpoint check
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  tcpSocket:     # TCP socket check
    port: 22
  exec:          # Command execution check
    command:
    - cat
    - /tmp/healthy
  
  # Timing parameters (applies to all probe types)
  initialDelaySeconds: 5    # Wait before first probe
  periodSeconds: 10         # How often to perform the probe
  timeoutSeconds: 1         # Timeout for each probe
  successThreshold: 1       # Consecutive successes required
  failureThreshold: 3       # Consecutive failures before marking as failed
```

## Common Probe Patterns

### 1. HTTP Endpoint Check

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    scheme: HTTP  # or HTTPS
  initialDelaySeconds: 3
  periodSeconds: 10
  timeoutSeconds: 1
  failureThreshold: 3
```

### 2. TCP Socket Check

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 5
  periodSeconds: 10
```

### 3. Command Execution Check

```yaml
livenessProbe:
  exec:
    command:
    - pgrep
    - "nginx"
  initialDelaySeconds: 5
  periodSeconds: 10
```

## Troubleshooting Probe Failures

### 1. Check Pod Status

```bash
# Get pod status
kubectl get pods

# Describe pod to see probe failures
kubectl describe pod <pod-name>

# Check container status
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'
```

### 2. Check Container Logs

```bash
# Get container logs
kubectl logs <pod-name>

# Get previous container logs if crashed
kubectl logs <pod-name> --previous

# Stream logs in real-time
kubectl logs -f <pod-name>
```

### 3. Debugging Probes

```bash
# Check probe configuration
kubectl get pod <pod-name> -o yaml | grep -A 20 'liveness\|readiness\|startup'

# Check kubelet logs on the node
# On the node:
journalctl -u kubelet | grep -i probe

# Check container's network connectivity
kubectl exec -it <pod-name> -- /bin/sh
# Inside container:
wget -qO- http://localhost:8080/healthz
```

## Common Probe Issues and Solutions

### 1. Liveness Probe Failing

**Symptoms:**
- Container restarts frequently
- `kubectl describe pod` shows `Liveness probe failed`

**Solutions:**
```yaml
# Increase initial delay
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30  # Increase if app takes time to start
  periodSeconds: 10
```

### 2. Readiness Probe Failing

**Symptoms:**
- Pod is not receiving traffic
- Endpoints not being added to the service

**Solutions:**
```yaml
# Adjust failure threshold
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  failureThreshold: 5      # Allow more failures before marking as not ready
  periodSeconds: 5
```

### 3. Startup Probe Configuration

**Use Case:** Applications with long initialization times

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30     # Retry for 5 minutes (30 * 10s)
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  timeoutSeconds: 1
```

## Advanced Probe Scenarios

### 1. Command-Based Liveness Check

```yaml
livenessProbe:
  exec:
    command:
    - sh
    - -c
    - 'pgrep -x "myapp" || exit 1'
  initialDelaySeconds: 5
  periodSeconds: 10
```

### 2. HTTP Headers and HTTPS

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8443
    scheme: HTTPS
    httpHeaders:
    - name: X-Custom-Header
      value: CustomValue
  initialDelaySeconds: 5
  periodSeconds: 10
```

### 3. Multiple Readiness Gates

```yaml
readinessGates:
- conditionType: "www.example.com/feature-1"
- conditionType: "www.example.com/feature-2"
```

## Best Practices

1. **Always Define Readiness Probes**
   - Prevents sending traffic to pods that aren't ready
   - Essential for rolling updates

2. **Use Startup Probes for Slow-Starting Containers**
   - Prevents unnecessary restarts during initialization
   - More reliable than long initial delays

3. **Keep Liveness Probes Lightweight**
   - Should not depend on external services
   - Should return quickly to avoid cascading failures

4. **Set Appropriate Timeouts and Thresholds**
   - Consider application response times
   - Account for temporary network issues

5. **Monitor Probe Failures**
   - Set up alerts for frequent probe failures
   - Log probe results for debugging

## Debugging Example: Failing Liveness Probe

1. **Check Pod Status**
   ```bash
   kubectl get pods
   # NAME                     READY   STATUS    RESTARTS   AGE
   # myapp-5d8f8c9f6c-abc12   0/1     Running   3          2m
   ```

2. **Describe the Pod**
   ```bash
   kubectl describe pod myapp-5d8f8c9f6c-abc12
   # Look for:
   # Liveness probe failed: HTTP probe failed with statuscode: 500
   # Liveness probe failed: Get http://10.244.0.5:8080/healthz: dial tcp 10.244.0.5:8080: connect: connection refused
   ```

3. **Check Container Logs**
   ```bash
   kubectl logs myapp-5d8f8c9f6c-abc12
   ```

4. **Debug Inside Container**
   ```bash
   kubectl exec -it myapp-5d8f8c9f6c-abc12 -- /bin/sh
   # Inside container:
   wget -qO- http://localhost:8080/healthz
   ps aux
   netstat -tuln
   ```

5. **Fix and Update**
   ```bash
   # Update deployment with fixed probe
   kubectl edit deployment myapp
   
   # Or apply changes from file
   kubectl apply -f fixed-deployment.yaml
   ```

## Monitoring Probe Status

### 1. Using kube-state-metrics

```bash
# Install kube-state-metrics
kubectl apply -f https://github.com/kubernetes/kube-state-metrics/raw/master/examples/standard/

# Query probe status
kubectl get --raw /api/v1/namespaces/kube-system/services/kube-state-metrics:http-metrics/proxy/metrics | grep kube_pod_container_status_ready
```

### 2. Prometheus Queries

```promql
# Containers with failing liveness probes
kube_pod_container_status_last_terminated_reason{reason="ContainerFailed"} == 1

# Containers not ready
kube_pod_container_status_ready == 0

# Containers with restarts
kube_pod_container_status_restarts_total > 0
```

## Security Considerations

1. **Probe Endpoint Security**
   - Use a dedicated port for health checks
   - Implement authentication if needed
   - Consider network policies to restrict access

2. **Resource Usage**
   - Probes consume resources
   - Set appropriate timeouts and intervals
   - Monitor probe-related resource usage

3. **Privilege Escalation**
   - Be cautious with `exec` probes
   - Use read-only filesystem for probe commands
   - Run probes with non-root users when possible

## Performance Impact

### 1. Probe Overhead
- Each probe creates network or process overhead
- Consider probe frequency and timeout values
- Monitor kubelet CPU usage

### 2. Optimizing Probes
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10        # Default is 10s
  timeoutSeconds: 1        # Default is 1s
  successThreshold: 1      # Default is 1
  failureThreshold: 3      # Default is 3
```

## Conclusion

Effective use of probes is crucial for maintaining application health and availability in Kubernetes. By understanding the different types of probes, their configuration options, and common troubleshooting techniques, you can ensure your applications are properly monitored and managed. Remember to:

1. Use all three probe types appropriately
2. Set meaningful timeouts and thresholds
3. Monitor probe failures and container restarts
4. Follow security best practices
5. Regularly test and validate your probe configurations

By implementing these best practices, you can create more resilient and self-healing applications in your Kubernetes clusters.

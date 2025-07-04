# 9.6 Troubleshooting Ingress

## Introduction to Ingress Troubleshooting

When working with Ingress in Kubernetes, you might encounter various issues. This guide provides a systematic approach to diagnosing and resolving common Ingress-related problems.

## Common Issues and Solutions

### 1. Ingress Controller Not Running

**Symptoms**:
- No response when accessing the Ingress IP/hostname
- No Ingress controller pods running

**Diagnosis**:
```bash
# Check if the Ingress controller is running
kubectl get pods -n ingress-nginx

# Check for any errors in the controller logs
kubectl logs -n ingress-nginx <ingress-controller-pod>
```

**Solutions**:
- Ensure the Ingress controller is properly installed
- Check resource quotas and limits
- Verify node selectors and tolerations

### 2. Service Not Reachable

**Symptoms**:
- 503 Service Temporarily Unavailable
- 404 Not Found errors

**Diagnosis**:
```bash
# Check if the service exists and has endpoints
kubectl get svc <service-name>
kubectl get endpoints <service-name>

# Verify the service selector matches pod labels
kubectl describe svc <service-name>
kubectl get pods --show-labels
```

**Solutions**:
- Ensure the service selector matches pod labels
- Verify the service port matches the container port
- Check if pods are in the Running state

### 3. Incorrect DNS Resolution

**Symptoms**:
- DNS resolution fails
- Host not found errors

**Diagnosis**:
```bash
# Verify DNS resolution from within the cluster
kubectl run -it --rm --image=curlimages/curl --restart=Never -- curl -v http://<service-name>.<namespace>.svc.cluster.local

# Check CoreDNS/CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

**Solutions**:
- Verify CoreDNS/CoreDNS is running
- Check /etc/resolv.conf in pods
- Validate network policies

## Step-by-Step Troubleshooting Guide

### 1. Verify Ingress Resource

```bash
# Get all Ingress resources
kubectl get ingress --all-namespaces

# Describe a specific Ingress
kubectl describe ingress <ingress-name> -n <namespace>
```

### 2. Check Ingress Controller Logs

```bash
# Get Ingress controller pods
kubectl get pods -n ingress-nginx

# View logs
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Follow logs in real-time
kubectl logs -f -n ingress-nginx <ingress-controller-pod>
```

### 3. Verify Services and Endpoints

```bash
# List all services
kubectl get svc --all-namespaces

# Check service details
kubectl describe svc <service-name> -n <namespace>

# Verify endpoints
kubectl get endpoints <service-name> -n <namespace>
```

### 4. Check Network Policies

```bash
# List all network policies
kubectl get networkpolicies --all-namespaces

# Describe a specific network policy
kubectl describe networkpolicy <policy-name> -n <namespace>
```

### 5. Test Connectivity

From outside the cluster:
```bash
# Get the external IP or hostname
kubectl get ingress -n <namespace>

# Test connectivity
curl -v http://<ingress-ip-or-hostname>/path
```

From inside the cluster:
```bash
# Run a temporary debug pod
kubectl run -it --rm --image=curlimages/curl --restart=Never -- sh

# Test connectivity to services
curl http://<service-name>.<namespace>.svc.cluster.local
```

## Common Error Messages and Fixes

### 1. "default backend - 404"
- **Cause**: No default backend is configured
- **Fix**: Configure a default backend or ensure all paths are correctly routed

### 2. "503 Service Temporarily Unavailable"
- **Cause**: No endpoints available for the service
- **Fix**: Check if pods are running and service selectors match pod labels

### 3. "Connection refused"
- **Cause**: The service is not listening on the specified port
- **Fix**: Verify the container port matches the service targetPort

### 4. "Certificate signed by unknown authority"
- **Cause**: Missing or invalid TLS certificate
- **Fix**: Ensure the TLS secret exists and is correctly referenced

## Advanced Troubleshooting

### 1. Enable Debug Logging

For NGINX Ingress Controller:
```yaml
# nginx-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
data:
  error-log-level: "debug"
```

### 2. Check Ingress Controller Configuration

```bash
# Get the current configuration
kubectl exec -n ingress-nginx <ingress-controller-pod> -- cat /etc/nginx/nginx.conf

# Check generated NGINX configuration
kubectl exec -n ingress-nginx <ingress-controller-pod> -- nginx -T
```

### 3. Network Policy Verification

```bash
# Check if network policies are blocking traffic
kubectl describe networkpolicies --all-namespaces

# Check if pods can communicate with each other
kubectl run -it --rm --image=alpine --restart=Never -- sh
# Inside the container:
apk add --no-cache curl
curl -v http://<service-name>.<namespace>.svc.cluster.local
```

## Best Practices for Troubleshooting

1. **Start Simple**
   - Begin with basic checks before diving deep
   - Verify the most common issues first

2. **Use Namespace Context**
   - Always specify the namespace when running commands
   - Use `-n` or `--namespace` flag

3. **Check Events**
   ```bash
   # Get events for a specific resource
   kubectl describe <resource-type> <resource-name> -n <namespace>
   
   # Get all events in a namespace
   kubectl get events -n <namespace>
   ```

4. **Verify Resource Availability**
   - Check CPU and memory usage
   - Verify node conditions
   - Check for eviction events

5. **Documentation**
   - Keep track of changes made
   - Document the troubleshooting process
   - Share findings with the team

## Common Pitfalls

1. **Mismatched Labels**
   - Ensure service selectors match pod labels exactly
   - Check for typos in label names and values

2. **Port Mismatches**
   - Verify container ports match service targetPorts
   - Check protocol (TCP/UDP) matches

3. **Network Policies**
   - Default deny-all policies can block traffic
   - Verify network policies allow the required traffic

4. **DNS Resolution**
   - Ensure CoreDNS/CoreDNS is running
   - Check /etc/resolv.conf in pods

## Next Steps

After resolving Ingress issues, consider:
1. Implementing monitoring and alerting
2. Setting up automated testing for Ingress configurations
3. Documenting common issues and solutions for your team
4. Regularly updating Ingress controller and related components

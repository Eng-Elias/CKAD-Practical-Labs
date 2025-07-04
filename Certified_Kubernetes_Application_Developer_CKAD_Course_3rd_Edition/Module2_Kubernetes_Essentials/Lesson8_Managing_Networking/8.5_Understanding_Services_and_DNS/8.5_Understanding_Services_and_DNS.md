# 8.5 Understanding Services and DNS

## CoreDNS in Kubernetes

### Overview
- Default DNS service in Kubernetes
- Runs as a cluster add-on
- Provides DNS-based service discovery

### CoreDNS Components
1. **CoreDNS Pods**
   - Run in the `kube-system` namespace
   - Managed by a Deployment or DaemonSet
   - Configured via a ConfigMap

2. **kube-dns Service**
   - Exposes CoreDNS to the cluster
   - Has a stable cluster IP

## DNS for Services

### A/AAAA Records
- Standard DNS A records for services
- Format: `<service-name>.<namespace>.svc.cluster.local`
- Example: `my-service.default.svc.cluster.local`

### SRV Records
- Used for named ports
- Format: `_port-name._port-protocol.<service-name>.<namespace>.svc.cluster.local`
- Example: `_http._tcp.my-service.default.svc.cluster.local`

## DNS Resolution in Pods

### Pod DNS Policy
- `ClusterFirst`: Default, queries cluster DNS first
- `Default`: Inherits node's DNS settings
- `None`: Uses custom DNS settings

### Pod DNS Config
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-example
spec:
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
  containers:
    - name: test
      image: nginx
```

## Service Discovery Patterns

### Environment Variables
- Injected at Pod creation
- Example: `MY_SERVICE_SERVICE_HOST`
- Limited to services created before the Pod

### DNS-Based Discovery
- Preferred method
- Dynamic and works across namespaces
- Example: `my-service.my-namespace.svc.cluster.local`

## Hands-On Examples

### 1. Verify CoreDNS Installation
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get svc -n kube-system kube-dns
```

### 2. Test DNS Resolution
```bash
# Run a temporary busybox pod
kubectl run -it --rm --restart=Never --image=busybox:1.28 dns-test -- /bin/sh

# Inside the container
nslookup kubernetes.default
nslookup my-service
nslookup my-service.my-namespace
nslookup my-service.my-namespace.svc.cluster.local
```

### 3. Check DNS Configuration
```bash
# View CoreDNS config
kubectl -n kube-system get configmap coredns -o yaml

# Check DNS resolution from a pod
kubectl exec -it my-pod -- cat /etc/resolv.conf
```

## DNS Performance and Optimization

### Caching
- Node-level DNS cache (NodeLocal DNSCache)
- Application-level caching

### Tuning
- Adjust `ndots` in resolv.conf
- Optimize DNS queries
- Use headless services when possible

## Troubleshooting DNS Issues

### Common Problems
1. **DNS Resolution Failing**
   - Check CoreDNS pods
   - Verify network policies
   - Test basic connectivity

2. **Slow DNS Resolution**
   - Check DNS cache settings
   - Review CoreDNS metrics
   - Consider NodeLocal DNSCache

3. **Incorrect DNS Records**
   - Verify service selectors
   - Check endpoint objects
   - Review CoreDNS logs

### Diagnostic Commands
```bash
# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check DNS configuration in a pod
kubectl exec -it my-pod -- cat /etc/resolv.conf

# Test DNS resolution from a pod
kubectl exec -it my-pod -- nslookup kubernetes.default

# Check service endpoints
kubectl get endpoints my-service
```

## Best Practices

### 1. Service Naming
- Use DNS-compliant names
- Be consistent with naming conventions
- Avoid special characters

### 2. DNS Configuration
- Use ClusterFirst DNS policy
- Configure proper search domains
- Consider DNS cache settings

### 3. Monitoring
- Monitor CoreDNS metrics
- Set up alerts for DNS resolution failures
- Review DNS query patterns

## Advanced Topics

### Headless Services
- For direct Pod access
- Returns multiple A/AAAA records
- Useful for stateful applications

### ExternalName Services
- Maps to external DNS names
- Acts as a CNAME record
- No selectors or endpoints

### Custom DNS Entries
- Using CoreDNS plugins
- Adding custom domain mappings
- Forwarding to external DNS servers

## Next Steps
In the next section, we'll explore NetworkPolicies for securing network traffic between pods and services.

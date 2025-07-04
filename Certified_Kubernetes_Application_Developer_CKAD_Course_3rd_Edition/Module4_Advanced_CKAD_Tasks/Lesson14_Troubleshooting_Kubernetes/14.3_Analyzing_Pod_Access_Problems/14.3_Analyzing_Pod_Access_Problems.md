# 14.3 Analyzing Pod Access Problems

## Understanding Kubernetes Networking

When troubleshooting pod access issues, it's essential to understand how Kubernetes networking works:

1. **Pod-to-Pod Communication**: All pods can communicate with all other pods without NAT
2. **Service Abstraction**: Services provide stable IP addresses and DNS names for pods
3. **Ingress**: Manages external access to services, typically HTTP/HTTPS
4. **Network Policies**: Define how pods communicate with each other and other network endpoints

## Common Access Issues and Solutions

### 1. Service Not Accessible

**Symptoms:**
- Cannot reach the service from within or outside the cluster
- Service exists but endpoints are not created

**Troubleshooting Steps:**

```bash
# Check if the service exists
kubectl get svc <service-name>

# Check service details
kubectl describe svc <service-name>

# Check endpoints - should show pod IPs
kubectl get endpoints <service-name>

# If no endpoints, check selector matches pod labels
kubectl get pods --show-labels
kubectl get svc <service-name> -o jsonpath='{.spec.selector}'
```

### 2. Port Forwarding Issues

**Symptoms:**
- `kubectl port-forward` fails or hangs
- Cannot access the forwarded port locally

**Troubleshooting Steps:**

```bash
# Check if the pod is running
kubectl get pods

# Try forwarding to a different local port
kubectl port-forward <pod-name> 8080:80

# Check for port conflicts
netstat -tuln | grep <port-number>

# Use verbose output for more details
kubectl port-forward --v=4 <pod-name> 8080:80
```

### 3. DNS Resolution Problems

**Symptoms:**
- Pods cannot resolve service names
- DNS lookups fail

**Troubleshooting Steps:**

```bash
# Check if CoreDNS/Kube-DNS is running
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check DNS service
kubectl get svc -n kube-system kube-dns

# Test DNS resolution from a pod
kubectl run -it --rm --image=busybox:1.28 dns-test -- nslookup kubernetes.default

# Check DNS configuration in the pod
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
```

## Network Policy Issues

### Checking Network Policies

```bash
# List all network policies
kubectl get networkpolicies --all-namespaces

# Describe a specific network policy
kubectl describe networkpolicy <policy-name> -n <namespace>

# Check if network policies are enforced
kubectl get networkpolicies --all-namespaces -o yaml
```

### Testing Network Connectivity

```bash
# Create a test pod
kubectl run -it --rm --image=nicolaka/netshoot test-pod -- /bin/bash

# Inside the container, test connectivity
curl http://<service-name>.<namespace>.svc.cluster.local
nslookup <service-name>
telnet <service-ip> <port>

# Test DNS resolution
nslookup kubernetes.default.svc.cluster.local
```

## Ingress Controller Issues

### Checking Ingress Resources

```bash
# List all ingresses
kubectl get ingress --all-namespaces

# Describe a specific ingress
kubectl describe ingress <ingress-name> -n <namespace>

# Check ingress controller logs
kubectl logs -n <ingress-namespace> <ingress-controller-pod>
```

### Common Ingress Issues

1. **Ingress Controller Not Running**
   ```bash
   # Check if the ingress controller pods are running
   kubectl get pods -n <ingress-namespace>
   ```

2. **Misconfigured Ingress Rules**
   ```bash
   # Check ingress rules
   kubectl get ingress -o yaml
   ```

3. **TLS/HTTPS Issues**
   ```bash
   # Check TLS secrets
   kubectl get secret -n <namespace>
   
   # View certificate details
   kubectl get secret <tls-secret> -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text
   ```

## Network Plugin Issues

### Checking CNI Plugin

```bash
# Check CNI plugin status
kubectl get pods -n kube-system -l k8s-app=cilium  # For Cilium
kubectl get pods -n kube-system -l k8s-app=calico-node  # For Calico
kubectl get pods -n kube-system -l k8s-app=flannel  # For Flannel

# Check CNI configuration
cat /etc/cni/net.d/*
```

### Common CNI Issues

1. **Node Network Not Ready**
   ```bash
   # Check node status
   kubectl get nodes
   
   # Check node conditions
   kubectl describe node <node-name> | grep -A 10 "Conditions:"
   ```

2. **IP Address Exhaustion**
   ```bash
   # Check pod IP addresses
   kubectl get pods -o wide --all-namespaces
   
   # Check IP allocation status (for Calico)
   kubectl get ipamblocks -o yaml
   ```

## Troubleshooting DNS Issues

### CoreDNS/Kube-DNS Debugging

```bash
# Check CoreDNS/Kube-DNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS/Kube-DNS service
kubectl get svc -n kube-system kube-dns

# Check CoreDNS configuration
kubectl get configmap -n kube-system coredns -o yaml

# Check DNS queries
kubectl run -it --rm --image=infoblox/dnstools:latest dnstools
# Inside the container:
# nslookup kubernetes.default
# dig kubernetes.default.svc.cluster.local
```

### Common DNS Issues

1. **DNS Resolution Failing**
   ```bash
   # Check DNS server in pod
   kubectl exec -it <pod-name> -- cat /etc/resolv.conf
   
   # Test DNS resolution from node
   kubectl debug node/<node-name> -it --image=busybox
   # Inside the container:
   # nslookup kubernetes.default
   ```

2. **Slow DNS Resolution**
   ```bash
   # Check DNS server response time
   kubectl run -it --rm --image=busybox:1.28 dns-test -- time nslookup kubernetes.default
   
   # Check for DNS timeouts in pod logs
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```

## Network Policy Debugging

### Testing Network Policies

```bash
# Create a test pod with network debugging tools
kubectl run -it --rm --image=nicolaka/netshoot test-pod -- /bin/bash

# Inside the container, test connectivity
# Test TCP connectivity
telnet <service-ip> <port>

# Test HTTP connectivity
curl -v http://<service-ip>:<port>

# Check network connections
netstat -tuln
ss -tuln
```

### Common Network Policy Issues

1. **Too Restrictive Policies**
   ```bash
   # Check network policies in the namespace
   kubectl get networkpolicies -n <namespace>
   
   # Temporarily allow all traffic for testing
   cat <<EOF | kubectl apply -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: allow-all
     namespace: <namespace>
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
     ingress:
     - {}
     egress:
     - {}
   EOF
   ```

2. **Missing Egress Rules**
   ```bash
   # Check if pods can reach external services
   kubectl run -it --rm --image=busybox:1.28 test-egress -- wget -qO- ifconfig.co
   ```

## Service Mesh Issues

### Istio Debugging

```bash
# Check Istio components
kubectl get pods -n istio-system

# Check Envoy proxy logs
kubectl logs <pod-name> -c istio-proxy -n <namespace>

# Check Istio configuration
kubectl get virtualservices -n <namespace>
kubectl get destinationrules -n <namespace>
kubectl get serviceentries -n <namespace>
```

### Linkerd Debugging

```bash
# Check Linkerd components
linkerd check

# Check proxy logs
kubectl logs <pod-name> -n <namespace> -c linkerd-proxy

# Check service profiles
kubectl get serviceprofiles -n <namespace>
```

## Firewall and Security Group Issues

### Checking Firewall Rules

```bash
# Check iptables rules (on the node)
sudo iptables-save | grep <service-ip>

# Check network policies
kubectl get networkpolicies --all-namespaces

# Check security groups (cloud provider specific)
# AWS: aws ec2 describe-security-groups
# GCP: gcloud compute firewall-rules list
# Azure: az network nsg list
```

### Common Firewall Issues

1. **NodePort Not Accessible**
   ```bash
   # Check NodePort service
   kubectl get svc <service-name> -o yaml | grep -i nodeport
   
   # Check if the port is open on the node
   nc -zv <node-ip> <node-port>
   ```

2. **LoadBalancer Not Working**
   ```bash
   # Check LoadBalancer service
   kubectl get svc <service-name> -o wide
   
   # Check cloud provider load balancer
   # AWS: aws elb describe-load-balancers
   # GCP: gcloud compute forwarding-rules list
   # Azure: az network lb list
   ```

## Performance Issues

### Network Performance Testing

```bash
# Install iperf3 in the cluster
kubectl create deploy iperf-server --image=networkstatic/iperf3
kubectl expose deploy iperf-server --port=5201 --target-port=5201

# Run a client to test bandwidth
kubectl run -it --rm --image=networkstatic/iperf3 iperf-client -- iperf3 -c iperf-server
```

### Common Performance Issues

1. **High Latency**
   ```bash
   # Check network latency between pods
   kubectl run -it --rm --image=busybox:1.28 test-latency -- ping <target-pod-ip>
   ```

2. **Low Throughput**
   ```bash
   # Test network throughput
   kubectl run -it --rm --image=busybox:1.28 test-throughput -- wget -O /dev/null http://<service-ip>:<port>/large-file
   ```

## Conclusion

Troubleshooting pod access problems in Kubernetes requires a systematic approach. Start by verifying basic connectivity, then move on to more complex scenarios involving services, ingress controllers, and network policies. Always check the following:

1. Are the pods running and healthy?
2. Are the services correctly configured with proper selectors?
3. Are there any network policies blocking traffic?
4. Is DNS resolution working correctly?
5. Are there any firewall or security group rules preventing access?

By following these steps and using the provided commands, you can effectively diagnose and resolve most pod access issues in your Kubernetes cluster.

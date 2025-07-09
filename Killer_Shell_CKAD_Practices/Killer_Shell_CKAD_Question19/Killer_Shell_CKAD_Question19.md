# CKAD Practice Question 19: Converting ClusterIP to NodePort Service

## Question
In the `jupyter` namespace, there is an Apache deployment with one replica named `jupyter-crew-deploy` and a ClusterIP service called `jupyter-crew-svc` that exposes it.

**Tasks**:
1. Change the service type from ClusterIP to NodePort
2. Configure it to be available on all nodes on port 30100
3. Test the NodePort service using the internal IP of all available nodes on port 30100
4. Determine on which node the service is reachable

**Requirements**:
- Service must be accessible via NodePort 30100
- Test connectivity using `curl`
- Verify the node where the port is running

## Solution

### Step 1: Check Current Resources
```bash
# List all resources in the jupyter namespace
kubectl -n jupyter get all

# Get detailed information about the service
kubectl -n jupyter describe svc jupyter-crew-svc

# Verify the current service type
kubectl -n jupyter get svc jupyter-crew-svc -o jsonpath='{.spec.type}'
```

### Step 2: Test Current Service (ClusterIP)
```bash
# Test the current ClusterIP service from a temporary pod
kubectl -n jupyter run test-curl --image=nginx:alpine --rm -it --restart=Never -- \
  sh -c "apk add --no-cache curl && curl http://jupyter-crew-svc:8080"
```

### Step 3: Update Service to NodePort
Edit the service to change its type and add NodePort configuration:

```bash
# Edit the service
kubectl -n jupyter edit svc jupyter-crew-svc
```

Modify the service specification to include:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jupyter-crew-svc
  namespace: jupyter
spec:
  type: NodePort  # Changed from ClusterIP
  ports:
  - port: 8080
    targetPort: 80
    nodePort: 30100  # Add this line
  selector:
    app: jupyter-crew  # Make sure this matches your pod labels
```

### Step 4: Verify the Changes
```bash
# Check the updated service
kubectl -n jupyter get svc jupyter-crew-svc

# Expected output should show:
# NAME               TYPE       CLUSTER-IP      PORT(S)          AGE
# jupyter-crew-svc   NodePort   10.96.XXX.XXX   8080:30100/TCP   XXm
```

### Step 5: Test NodePort Connectivity
```bash
# Get node internal IPs
kubectl get nodes -o wide

# Test connectivity to NodePort on each node
# Replace <node-ip> with the actual node IP from the previous command
curl http://<node-ip>:30100

# To test on all nodes in a loop
for NODE_IP in $(kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'); do
  echo "Testing $NODE_IP..."
  curl -s -m 2 http://$NODE_IP:30100 || echo "Failed to connect to $NODE_IP"
done
```

## Explanation

### Key Concepts
1. **Service Types**:
   - **ClusterIP**: Exposes the service on an internal IP in the cluster (default)
   - **NodePort**: Exposes the service on each Node's IP at a static port (NodePort)
   - **LoadBalancer**: Creates an external load balancer in supported environments

2. **NodePort Range**:
   - Default range: 30000-32767
   - Must be within this range unless the API server is configured otherwise

3. **Port Definitions**:
   - `port`: The service port (e.g., 8080)
   - `targetPort`: The container port (e.g., 80)
   - `nodePort`: The port on each node (e.g., 30100)

### Why This Solution Works
- Changing the service type to NodePort makes it accessible from outside the cluster
- The specified nodePort (30100) is within the valid range
- The service routes traffic to the correct pods using the selector
- The port mapping is correctly configured from nodePort → port → targetPort

### Exam Tips
1. **Service Configuration**:
   - Always verify the selector matches the pod labels
   - Ensure the targetPort matches the container's listening port
   - Remember the NodePort range (30000-32767)

2. **Troubleshooting**:
   - Use `kubectl describe svc` to check service details
   - Verify endpoints with `kubectl get endpoints`
   - Check node firewall rules if NodePort is not accessible

3. **Best Practices**:
   - Prefer ClusterIP for internal service communication
   - Use NodePort only when external access is required
   - Consider using an Ingress controller for production HTTP/HTTPS traffic

### Common Mistakes to Avoid
- Specifying a NodePort outside the valid range
- Mismatched selectors between service and pods
- Forgetting to check node firewall rules
- Not verifying the service type after changes
- Confusing port, targetPort, and nodePort

## Additional Practice
1. Create a multi-node cluster and test NodePort accessibility across different nodes
2. Configure a deployment with multiple containers and expose them via different NodePorts
3. Implement network policies to restrict NodePort access
4. Set up a LoadBalancer service in a cloud environment
5. Create an Ingress resource to route traffic to multiple services

## Related Commands
```bash
# Get service details in YAML format
kubectl -n <namespace> get svc <service-name> -o yaml

# Check service endpoints
kubectl -n <namespace> get endpoints <service-name>

# View service events
kubectl -n <namespace> get events --field-selector involvedObject.name=<service-name>

# Check kube-proxy logs (on worker nodes)
kubectl -n kube-system logs -l k8s-app=kube-proxy

# Check node ports in use
kubectl get svc --all-namespaces | grep NodePort
```

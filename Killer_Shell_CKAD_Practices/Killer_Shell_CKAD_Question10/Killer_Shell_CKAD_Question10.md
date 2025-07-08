# CKAD Practice Question 10: Creating a ClusterIP Service and Testing Connectivity

## Question
Team Pluto needs a new cluster-internal service with the following requirements:

**Tasks**:
1. Create a pod named `project-plt-6cc-api` in the `pluto` namespace with:
   - Image: `nginx:1.17.3-alpine`
   - Label: `project=plt-6cc-api`

2. Create a ClusterIP service named `project-plt-6cc-svc` in the `pluto` namespace that:
   - Selects the pod using the label `project=plt-6cc-api`
   - Exposes port 3333 and forwards to port 80 on the pod

3. Test the service by:
   - Creating a temporary pod to curl the service
   - Saving the response to `/opt/course/10/response.html`
   - Saving the pod logs to `/opt/course/10/pod-logs.txt`

**Important Notes**:
- The service should only be accessible within the cluster (ClusterIP)
- The test pod should be temporary and cleaned up after use
- Ensure proper permissions to write to the specified files

## Solution

### Step 1: Create the Pod
```bash
# Create the pod with the specified label
kubectl run project-plt-6cc-api \
  --image=nginx:1.17.3-alpine \
  --labels=project=plt-6cc-api \
  -n pluto

# Verify the pod is running
kubectl get pod project-plt-6cc-api -n pluto
```

### Step 2: Create the ClusterIP Service
```bash
# Create the ClusterIP service
kubectl create service clusterip project-plt-6cc-svc \
  --tcp=3333:80 \
  --dry-run=client \
  -o yaml > service.yaml

# Add the selector to the service
cat <<EOF >> service.yaml
  selector:
    project: plt-6cc-api
EOF

# Apply the service
kubectl apply -f service.yaml -n pluto

# Verify the service
kubectl get svc project-plt-6cc-svc -n pluto
```

### Step 3: Test the Service
```bash
# Create a temporary pod to test the service
kubectl run test-curl \
  --image=nginx:1.17.3-alpine \
  --restart=Never \
  --rm -it \
  -- /bin/sh -c 'curl -s http://project-plt-6cc-svc.pluto:3333' > /opt/course/10/response.html

# Save the pod logs
kubectl logs project-plt-6cc-api -n pluto > /opt/course/10/pod-logs.txt

# Verify the files were created
ls -l /opt/course/10/
```

## Explanation

### Key Concepts
1. **ClusterIP Service**: Internal service that exposes the pod within the cluster
2. **Labels and Selectors**: Used to associate services with pods
3. **Port Forwarding**: Maps service ports to container ports
4. **Temporary Pods**: Useful for testing without permanent cluster changes

### Why This Solution Works
- The pod is created with the required label for service discovery
- The service uses ClusterIP type for internal cluster access
- Port 3333 on the service forwards to port 80 on the pod
- The test pod verifies connectivity and saves the response
- Pod logs capture the request details for verification

### Exam Tips
1. **Service Types**: Remember ClusterIP is the default and only accessible within the cluster
2. **Label Matching**: Ensure service selectors match pod labels exactly
3. **Port Mapping**: Format is `service-port:container-port`
4. **Temporary Pods**: Use `--rm` to automatically clean up test pods
5. **Namespace Awareness**: Always specify the namespace with `-n`

### Common Mistakes to Avoid
- Forgetting to include the namespace in service DNS names
- Mismatched labels between service selectors and pod labels
- Incorrect port mapping in the service definition
- Not cleaning up temporary resources
- Missing file permissions when writing to host paths

## Additional Practice
1. Create a NodePort service instead of ClusterIP
2. Add multiple ports to the service
3. Configure session affinity on the service
4. Create an external service with a static IP
5. Set up network policies to control access to the service

## Related Commands
```bash
# Get detailed service information
kubectl describe svc project-plt-6cc-svc -n pluto

# Check service endpoints
kubectl get endpoints project-plt-6cc-svc -n pluto

# Port-forward to test the service locally
kubectl port-forward svc/project-plt-6cc-svc 8080:3333 -n pluto

# Get service details in YAML format
kubectl get svc project-plt-6cc-svc -n pluto -o yaml

# Delete the service and pod
kubectl delete svc project-plt-6cc-svc -n pluto
kubectl delete pod project-plt-6cc-api -n pluto
```

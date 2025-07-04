# 14.1 Determining a Troubleshooting Strategy

## Understanding the Kubernetes Resource Creation Flow

When you create resources in Kubernetes, the request follows a specific flow:

1. **API Server Interaction**
   - Your `kubectl` command sends a request to the API server
   - The API server validates and processes the request

2. **etcd Storage**
   - The resource definition is written to etcd
   - This is the source of truth for the cluster state

3. **Scheduling**
   - The scheduler identifies a suitable node for the pod
   - The decision is based on resource requirements, node selectors, etc.

4. **Node Execution**
   - The kubelet on the target node pulls the container image
   - The container runtime starts the container
   - The kubelet monitors the container's state

## Key Troubleshooting Commands

### 1. `kubectl get`
```bash
# Check the current state of resources
kubectl get pods
kubectl get services
kubectl get deployments
```

### 2. `kubectl describe`
```bash
# Get detailed information about a resource
kubectl describe pod <pod-name>
kubectl describe service <service-name>
```

### 3. `kubectl logs`
```bash
# View container logs
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>

# View logs from a specific container in a multi-container pod
kubectl logs <pod-name> -c <container-name>
```

## Common Pod States and Their Meanings

| State | Description | Common Causes |
|-------|-------------|---------------|
| Pending | Pod is waiting to be scheduled | Insufficient resources, scheduling constraints |
| ContainerCreating | Pod is being created | Image pull in progress, volume mounting |
| Running | Pod is running | Normal state |
| CrashLoopBackOff | Container is repeatedly crashing | Application error, misconfiguration |
| ImagePullBackOff | Failed to pull container image | Invalid image name, authentication issues |
| ErrImagePull | Error pulling the image | Network issues, invalid image reference |
| Completed | Container has run to completion | Normal for Jobs, indicates success |
| Error | Container failed to start | Application error, missing dependencies |
| Unknown | State cannot be determined | Node communication issues |

## Step-by-Step Troubleshooting Approach

1. **Check Pod Status**
   ```bash
   kubectl get pods
   ```

2. **Examine Pod Details**
   ```bash
   kubectl describe pod <pod-name>
   ```
   - Look at `Events` section for warnings or errors
   - Check container status and conditions

3. **Check Container Logs**
   ```bash
   kubectl logs <pod-name>
   ```

4. **Verify Resource Availability**
   ```bash
   # Check node resources
   kubectl describe nodes
   
   # Check resource quotas
   kubectl describe quota
   ```

5. **Inspect Cluster Events**
   ```bash
   kubectl get events --sort-by='.metadata.creationTimestamp'
   ```

## Common Issues and Solutions

### Issue: Pod Stuck in Pending State
1. Check node resources:
   ```bash
   kubectl describe nodes
   ```
2. Check for taints and tolerations:
   ```bash
   kubectl describe node <node-name> | grep -i taint
   ```
3. Check resource quotas:
   ```bash
   kubectl describe quota
   ```

### Issue: Container Failing to Start
1. Check container logs:
   ```bash
   kubectl logs <pod-name> --previous
   ```
2. Verify image name and pull policy:
   ```bash
   kubectl get pod <pod-name> -o yaml | grep -i image
   ```
3. Check image pull secrets:
   ```bash
   kubectl get secrets
   ```

### Issue: CrashLoopBackOff
1. Check container logs:
   ```bash
   kubectl logs <pod-name> --previous
   ```
2. Check container exit code:
   ```bash
   kubectl describe pod <pod-name> | grep -i exit
   ```
3. Check resource limits:
   ```bash
   kubectl describe pod <pod-name> | grep -i -A 5 resources
   ```

## Debugging Techniques

### 1. Execute Commands in a Running Container
```bash
kubectl exec -it <pod-name> -- /bin/sh
```

### 2. Port Forwarding for Local Access
```bash
kubectl port-forward <pod-name> <local-port>:<container-port>
```

### 3. Get Shell Access to a Node
```bash
# For Minikube
minikube ssh

# For other clusters
kubectl get nodes
kubectl debug node/<node-name> -it --image=busybox
```

### 4. Check Network Connectivity
```bash
# Create a temporary debug pod
kubectl run -it --rm --restart=Never debug --image=nicolaka/netshoot -- /bin/sh

# Inside the container, test connectivity
curl <service-name>.<namespace>.svc.cluster.local
nslookup <service-name>
```

## Troubleshooting Workflow

1. **Identify the Problem**
   - What is the symptom? (pod not starting, service not accessible, etc.)
   - When did it start happening?
   - What changed recently?

2. **Gather Information**
   - Collect logs and events
   - Check resource utilization
   - Review recent changes (deployments, config changes)

3. **Form a Hypothesis**
   - Based on the information, what could be the cause?
   - What tests can confirm or rule out possibilities?

4. **Test and Verify**
   - Make one change at a time
   - Verify if the change had the desired effect
   - Document the process

5. **Implement the Fix**
   - Apply the solution
   - Verify the issue is resolved
   - Document the resolution

## Best Practices

1. **Use Labels and Selectors**
   - Consistently label your resources
   - Makes it easier to query and manage related resources

2. **Implement Logging and Monitoring**
   - Use tools like Prometheus and Grafana
   - Set up alerts for critical conditions

3. **Use Namespaces**
   - Isolate resources by environment or team
   - Makes it easier to manage and troubleshoot

4. **Document Your Infrastructure**
   - Keep documentation up to date
   - Include troubleshooting steps for common issues

5. **Regularly Test Recovery Procedures**
   - Practice restoring from backups
   - Test failover procedures

## Example: Troubleshooting a Failing Deployment

1. **Check Deployment Status**
   ```bash
   kubectl get deployments
   kubectl describe deployment <deployment-name>
   ```

2. **Check ReplicaSet**
   ```bash
   kubectl get replicasets
   kubectl describe replicaset <replicaset-name>
   ```

3. **Check Pods**
   ```bash
   kubectl get pods -l app=<app-label>
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

4. **Check Events**
   ```bash
   kubectl get events --sort-by='.metadata.creationTimestamp'
   ```

5. **Check Services and Endpoints**
   ```bash
   kubectl get services
   kubectl get endpoints
   kubectl describe service <service-name>
   ```

By following this structured approach, you can systematically identify and resolve issues in your Kubernetes cluster.

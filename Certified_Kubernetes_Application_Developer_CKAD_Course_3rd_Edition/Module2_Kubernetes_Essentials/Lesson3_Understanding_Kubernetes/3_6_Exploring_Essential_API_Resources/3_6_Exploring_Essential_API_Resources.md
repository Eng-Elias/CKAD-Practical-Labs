# 3.6 Exploring Essential API Resources

## Understanding API Resources in Kubernetes

### What are API Resources?
- Fundamental building blocks in Kubernetes
- Represent the state of your cluster
- Managed through the Kubernetes API
- Follow a declarative model

### Core API Resources

#### 1. Pods
- Smallest deployable units in Kubernetes
- Can contain one or more containers
- Share network and storage
- Ephemeral by nature

#### 2. Controllers
- **Deployment**: Manages stateless applications
- **StatefulSet**: Manages stateful applications
- **DaemonSet**: Ensures all nodes run a copy of a pod
- **Job**: Creates one or more pods to complete a task
- **CronJob**: Runs jobs on a time-based schedule

#### 3. Services
- **ClusterIP**: Exposes the service on a cluster-internal IP
- **NodePort**: Exposes the service on each Node's IP at a static port
- **LoadBalancer**: Provisions an external load balancer
- **ExternalName**: Maps the service to a DNS name

#### 4. Configuration
- **ConfigMap**: Configuration data in key-value pairs
- **Secret**: Sensitive configuration data
- **ResourceQuota**: Limits resource consumption per namespace
- **LimitRange**: Constrains resource allocations

#### 5. Storage
- **PersistentVolume (PV)**: Storage resource in the cluster
- **PersistentVolumeClaim (PVC)**: Request for storage
- **StorageClass**: Defines storage "classes"

## Working with API Resources

### Viewing API Resources
```bash
# List all API resources
kubectl api-resources

# List all resources in a specific API group
kubectl api-resources --api-group=apps

# Get detailed information about a resource
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
```

### Common Operations

#### Create Resources
```bash
# Create from YAML
kubectl apply -f resource.yaml

# Create from command line
kubectl create deployment nginx --image=nginx
```

#### Get Resource Information
```bash
# Get basic information
kubectl get <resource>

# Get detailed information
kubectl describe <resource> <name>

# Get output in different formats
kubectl get <resource> -o yaml
kubectl get <resource> -o json
kubectl get <resource> -o wide
```

#### Update Resources
```bash
# Update from YAML
kubectl apply -f updated-resource.yaml

# Scale a deployment
kubectl scale deployment/nginx --replicas=3

# Update container image
kubectl set image deployment/nginx nginx=nginx:1.19.0
```

#### Delete Resources
```bash
# Delete by filename
kubectl delete -f resource.yaml

# Delete by resource type and name
kubectl delete <resource> <name>

# Delete all resources in a namespace
kubectl delete all --all -n <namespace>
```

## API Groups and Versions

### Core Group (Legacy)
- Path: `/api/v1`
- Contains fundamental resources like Pods, Services, Nodes

### Named Groups
- Path: `/apis/<group>/<version>`
- Examples:
  - `apps/v1` (Deployments, StatefulSets)
  - `networking.k8s.io/v1` (Ingress, NetworkPolicy)
  - `batch/v1` (Jobs, CronJobs)
  - `rbac.authorization.k8s.io/v1` (Roles, RoleBindings)

### Checking API Versions
```bash
# List all API groups and versions
kubectl api-versions

# View API resources for a specific group/version
kubectl api-resources --api-group=apps
```

## Custom Resource Definitions (CRDs)

### What are CRDs?
- Extend the Kubernetes API
- Define custom resources
- Enable custom controllers to manage them

### Example CRD
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

## Best Practices

### Resource Organization
- Use namespaces to organize resources
- Apply consistent labels and annotations
- Use resource quotas and limits

### Configuration Management
- Store configurations in version control
- Use Kustomize or Helm for complex configurations
- Implement GitOps workflows

### Security
- Apply least privilege principle
- Use RBAC effectively
- Regularly audit resource usage

## Common kubectl Commands

```bash
# Get resources with more details
kubectl get <resource> -o wide

# Watch resources in real-time
kubectl get <resource> -w

# Port-forward to a pod
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Execute a command in a pod
kubectl exec -it <pod-name> -- /bin/sh

# View pod logs
kubectl logs -f <pod-name>

# View events
kubectl get events --sort-by='.metadata.creationTimestamp'
```

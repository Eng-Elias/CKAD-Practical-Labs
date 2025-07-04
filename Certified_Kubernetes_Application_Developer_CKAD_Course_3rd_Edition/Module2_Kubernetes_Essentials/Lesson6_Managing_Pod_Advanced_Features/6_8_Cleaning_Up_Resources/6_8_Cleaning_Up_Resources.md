# 6.8 Cleaning Up Resources

## Introduction to Resource Cleanup

Properly cleaning up Kubernetes resources is essential for maintaining cluster health, controlling costs, and preventing resource leaks. This section covers various methods to clean up different types of resources in a Kubernetes cluster.

## Basic Cleanup Commands

### Deleting Individual Resources
```bash
# Delete a pod
kubectl delete pod <pod-name>

# Delete a deployment
kubectl delete deployment <deployment-name>

# Delete a service
kubectl delete service <service-name>

# Delete a configmap
kubectl delete configmap <configmap-name>

# Delete a secret
kubectl delete secret <secret-name>
```

### Deleting Multiple Resources
```bash
# Delete all resources of a specific type
kubectl delete pods --all
kubectl delete services --all
kubectl delete deployments --all
kubectl delete statefulsets --all
kubectl delete daemonsets --all
kubectl delete jobs --all
kubectl delete cronjobs --all
kubectl delete pvc --all
kubectl delete pv --all
```

## Namespace Cleanup

### Delete All Resources in a Namespace
```bash
# Delete all resources in a namespace (except the namespace itself)
kubectl delete all --all -n <namespace>

# Delete all resources including those not managed by 'all' (be careful!)
kubectl delete all,secret,pvc,configmap,serviceaccount,role,rolebinding \
  --all -n <namespace>
```

### Delete a Namespace and All Its Resources
```bash
# Delete a namespace and all resources within it
kubectl delete namespace <namespace>
```

## Selective Cleanup

### Using Labels
```bash
# Delete resources with a specific label
kubectl delete pods -l app=myapp
kubectl delete all -l environment=staging
```

### Using Field Selectors
```bash
# Delete pods in a specific phase
kubectl delete pods --field-selector=status.phase==Failed

# Delete completed jobs
kubectl delete jobs --field-selector=status.successful=1
```

## Advanced Cleanup Scenarios

### Orphaned Resources
```bash
# Find and delete pods in 'Terminating' state
kubectl get pods --field-selector=status.phase=Failed -o name | xargs kubectl delete

# Force delete a stuck namespace (use with caution)
kubectl get namespace <namespace> -o json \
  | tr -d "\n" | sed "s/\"finalize\":\\[[^]]*\]/\"finalize\": []/" \
  | kubectl replace --raw "/api/v1/namespaces/<namespace>/finalize" -f -
```

### Cleaning Up Persistent Volumes
```bash
# Delete all PVCs in a namespace
kubectl delete pvc --all -n <namespace>

# Delete all PVs that are not bound to any PVC
kubectl get pv | grep -v "bound" | grep -v "NAME" | awk '{print $1}' | xargs kubectl delete pv
```

## Automated Cleanup

### Using TTL Controller for Finished Resources
```yaml
# For Jobs (Kubernetes 1.21+)
apiVersion: batch/v1
kind: Job
metadata:
  name: cleanup-job
spec:
  ttlSecondsAfterFinished: 3600  # Clean up 1 hour after completion
  template:
    spec:
      containers:
      - name: cleaner
        image: busybox:1.28
        command: ["sh", "-c", "echo 'Cleaning up...' && sleep 30"]
      restartPolicy: Never
```

### Using Kubernetes CronJob for Regular Cleanup
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-cron
spec:
  schedule: "0 3 * * *"  # Run daily at 3 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - |
              # Delete failed pods
              kubectl delete pods --field-selector=status.phase=Failed --all-namespaces
              # Delete completed jobs older than 1 day
              kubectl delete jobs --field-selector=status.successful=1 --all-namespaces \
                --ignore-not-found=true --now=true --wait=false
          restartPolicy: OnFailure
```

## Best Practices for Resource Cleanup

1. **Use Namespaces**
   - Group related resources in namespaces
   - Makes cleanup easier and more organized

2. **Label Your Resources**
   - Use consistent labels for resources
   - Enables selective cleanup

3. **Automate Cleanup**
   - Use TTL controllers for temporary resources
   - Set up CronJobs for regular maintenance

4. **Monitor Resource Usage**
   - Set up alerts for resource consumption
   - Regularly review and clean up unused resources

5. **Document Cleanup Procedures**
   - Document cleanup processes for your team
   - Include cleanup in your CI/CD pipelines

## Common Cleanup Scenarios

### Scenario 1: Development Environment Reset
```bash
# Delete all resources in the dev namespace
kubectl delete all --all -n dev

# Delete all PVCs in the dev namespace
kubectl delete pvc --all -n dev

# Reset the database
kubectl delete job db-reset -n dev
kubectl apply -f db-reset-job.yaml -n dev
```

### Scenario 2: CI/CD Pipeline Cleanup
```bash
# Clean up resources from failed builds
kubectl delete all -l build=$BUILD_ID -n ci-cd

# Delete namespaces older than 1 day
kubectl get ns --field-selector=status.phase=Terminating -o name | \
  xargs -I {} kubectl get {} -o jsonpath='{.metadata.name} {.metadata.creationTimestamp}' | \
  awk '$2 <= "'$(date -d '1 day ago' -Ins --utc | sed 's/+0000/Z/')'" {print $1}' | \
  xargs kubectl delete ns
```

### Scenario 3: Cluster-wide Cleanup
```bash
# Clean up completed jobs in all namespaces
kubectl get jobs --all-namespaces --field-selector=status.successful=1 -o json \
  | jq -r '.items[] | "\(.metadata.namespace) \(.metadata.name)"' \
  | xargs -n2 sh -c 'kubectl delete job -n $0 $1'

# Clean up failed pods in all namespaces
kubectl delete pods --field-selector=status.phase=Failed --all-namespaces
```

## Troubleshooting Cleanup Issues

### Stuck in Terminating State
```bash
# Force delete a namespace
kubectl get namespace <namespace> -o json \
  | tr -d "\n" | sed "s/\"finalize\":\\[[^]]*\]/\"finalize\": []/" \
  | kubectl replace --raw "/api/v1/namespaces/<namespace>/finalize" -f -
```

### Orphaned Resources
```bash
# Find resources without owner references
kubectl api-resources --verbs=list --namespaced -o name \
  | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <namespace> \
  | grep -v "^NAMESPACE" | awk '$3=="" {print $0}'
```

## Next Steps
- Learn about Kubernetes Garbage Collection
- Explore Kubernetes Operators for custom resource management
- Understand Kubernetes Finalizers
- Study Backup and Restore strategies for Kubernetes

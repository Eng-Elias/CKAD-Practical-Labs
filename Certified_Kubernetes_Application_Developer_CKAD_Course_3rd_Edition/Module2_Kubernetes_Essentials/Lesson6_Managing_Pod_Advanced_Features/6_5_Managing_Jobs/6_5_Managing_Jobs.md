# 6.5 Managing Jobs

## Introduction to Kubernetes Jobs

Jobs in Kubernetes are designed to run pods to completion for batch processing or one-time tasks. Unlike regular pods managed by Deployments or ReplicaSets, Jobs ensure that a specified number of pods complete successfully.

### Key Characteristics
- **Completion Guarantee**: Ensures a pod runs to completion
- **Retry Mechanism**: Handles pod failures by retrying
- **Parallel Execution**: Supports running multiple pods in parallel
- **Cleanup**: Can be configured to clean up after completion

## Job Types

### 1. Non-Parallel Jobs
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

### 2. Parallel Jobs with Fixed Completion Count
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: process-item-
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - name: worker
        image: busybox:1.28
        command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
        env:
        - name: ITEM
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      restartPolicy: Never
```

### 3. Parallel Jobs with Work Queue
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: work-queue
spec:
  completions: 1
  parallelism: 3
  template:
    metadata:
      name: work-queue
    spec:
      containers:
      - name: worker
        image: busybox:1.28
        command: ["sh", "-c", "echo Processing work item $(($RANDOM%100)) && sleep 5"]
      restartPolicy: Never
```

## Job Management Commands

### Creating Jobs
```bash
# Create a job from YAML
kubectl create -f job.yaml

# Create job imperatively
kubectl create job my-job --image=busybox -- /bin/sh -c "echo Hello from Kubernetes && sleep 5"
```

### Monitoring Jobs
```bash
# List all jobs
kubectl get jobs

# Get detailed information
kubectl describe job/my-job

# View job logs
kubectl logs job/my-job

# Watch job status
kubectl get jobs -w
```

### Cleaning Up
```bash
# Delete a job (pods remain unless --cascade=orphan is used)
kubectl delete job/my-job

# Delete job and its pods
kubectl delete jobs --all

# Clean up completed jobs
kubectl delete jobs --field-selector=status.successful=1
```

## Advanced Job Features

### TTL Controller for Finished Jobs
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100  # Clean up 100 seconds after completion
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

### Job Backoff Limit
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-with-backoff
spec:
  backoffLimit: 6  # Number of retries before marking as failed
  template:
    spec:
      containers:
      - name: example
        image: busybox:1.28
        command: ["sh", "-c", "exit 1"]  # This will fail
      restartPolicy: Never
```

### Active Deadline
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-with-deadline
spec:
  activeDeadlineSeconds: 60  # Job will be terminated after 60 seconds
  template:
    spec:
      containers:
      - name: sleeper
        image: busybox:1.28
        command: ["sh", "-c", "sleep 120"]
      restartPolicy: Never
```

## Real-world Examples

### Database Migration Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  labels:
    app: db-migration
spec:
  template:
    metadata:
      name: db-migration
    spec:
      containers:
      - name: migrator
        image: myapp-db-migrator:1.0.0
        env:
        - name: DB_HOST
          value: "postgres"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
      restartPolicy: Never
  backoffLimit: 2
```

### Data Processing Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  completions: 10
  parallelism: 3
  template:
    spec:
      containers:
      - name: processor
        image: data-processor:2.1
        resources:
          requests:
            cpu: "500m"
            memory: "128Mi"
          limits:
            cpu: "1000m"
            memory: "256Mi"
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
      restartPolicy: Never
```

## Troubleshooting Jobs

### Common Issues

#### Job Not Starting
```bash
# Check for pod creation issues
kubectl get events --sort-by=.metadata.creationTimestamp

# Check quota limits
kubectl describe namespace default
```

#### Job Failing
```bash
# Check job status
kubectl describe job/my-job

# View logs from the last failed pod
kubectl logs job/my-job --tail=50

# View logs from a specific pod
kubectl logs <pod-name>
```

#### Job Stuck in "Running" State
```bash
# Check pod status
kubectl get pods -l job-name=my-job

# Check pod logs
kubectl logs <pod-name>

# Check for node issues
kubectl describe node <node-name>
```

## Best Practices

1. **Resource Management**
   - Always set resource requests and limits
   - Use appropriate backoff limits

2. **Error Handling**
   - Implement proper error handling in your application
   - Use appropriate restart policies

3. **Cleanup**
   - Set `ttlSecondsAfterFinished` for automatic cleanup
   - Regularly clean up completed jobs

4. **Monitoring**
   - Set up monitoring for job status
   - Configure alerts for job failures

5. **Idempotency**
   - Design jobs to be idempotent
   - Handle partial failures gracefully

## Next Steps
- Learn about CronJobs for scheduled tasks
- Explore Pod Templates and Controllers
- Understand Pod Lifecycle and Hooks
- Study Kubernetes Operators for complex job orchestration

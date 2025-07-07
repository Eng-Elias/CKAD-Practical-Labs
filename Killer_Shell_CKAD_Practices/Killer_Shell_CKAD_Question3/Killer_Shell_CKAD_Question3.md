# CKAD Practice Question 3: Creating a Kubernetes Job with Parallelism

## Question
**Task:** Create a Kubernetes Job with the following specifications:
- **Namespace:** `neptune`
- **Job Name:** `neb-new-job`
- **Container Name:** `neb-new-job-container`
- **Image:** `busybox:1.31.0`
- **Command:** `sh -c "sleep 2 && echo done"`
- **Completions:** 3 (run a total of three times)
- **Parallelism:** 2 (execute two runs in parallel)
- **Labels:** Each pod created by the job should have the label `id=awesome-job`

## Solution

### Method 1: Using kubectl create with --dry-run

1. **Create the job manifest**:
```bash
kubectl create job neb-new-job \
  --image=busybox:1.31.0 \
  -n neptune \
  --dry-run=client \
  -o yaml > job.yaml
```

2. **Edit the generated YAML** to add all required specifications:

```bash
vim job.yaml
```

change the following:
- change job name from neb-new-job to neb-new-job
- change container name from neb-new-job-container to neb-new-job-container
- change image from busybox:1.31.0 to busybox:1.31.0
- change command from sh -c "sleep 2 && echo done" to sh -c "sleep 2 && echo done"
- change completions from 3 to 3
- change parallelism from 2 to 2
- change labels from id=awesome-job to id=awesome-job

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: neb-new-job
  namespace: neptune
  labels:
    id: awesome-job
spec:
  completions: 3
  parallelism: 2
  template:
    metadata:
      labels:
        id: awesome-job
    spec:
      containers:
      - name: neb-new-job-container
        image: busybox:1.31.0
        command: ["sh", "-c"]
        args: ["sleep 2 && echo done"]
      restartPolicy: Never
```

3. **Create the job**:
```bash
kubectl apply -f job.yaml
```

4. **Verify the job and pods**:
```bash
# Check job status
kubectl get jobs -n neptune

# Check pods created by the job
kubectl get pods -n neptune -l job-name=neb-new-job

# View job logs
kubectl logs -n neptune -l job-name=neb-new-job --all-containers=true
```

### Method 2: Directly create the job
```bash
kubectl create job neb-new-job \
  --image=busybox:1.31.0 \
  -n neptune \
  --labels=id=awesome-job \
  -- /bin/sh -c "sleep 2 && echo done"

# Update the job to set completions and parallelism
kubectl patch job neb-new-job -n neptune -p '{"spec":{"completions":3,"parallelism":2}}'
```

## Explanation

### Key Concepts
1. **Kubernetes Jobs**: Create one or more pods and ensure that a specified number of them successfully terminate.
2. **Completions**: The number of successful completions required to mark the job as complete.
3. **Parallelism**: The maximum number of pods the job should run in parallel.
4. **Labels**: Key-value pairs that can be used to organize and select subsets of resources.

### Why This Solution Works
- The job is created in the `neptune` namespace with the specified name and container details.
- Setting `completions: 3` ensures the job runs three times.
- Setting `parallelism: 2` allows two pods to run simultaneously.
- The label `id: awesome-job` is applied to all pods created by the job.
- The command `sleep 2 && echo done` is executed in each container.

### Exam Tips
1. **Use `--dry-run`**: Quickly generate YAML manifests with `--dry-run=client -o yaml`.
2. **Check Documentation**: Use `kubectl explain job.spec` to understand all available fields.
3. **Verify Resources**: Always check if namespaces exist before creating resources.
4. **Label Resources**: Use meaningful labels for better resource management.
5. **Parallelism vs Completions**: Understand the difference - completions is the total number of successful runs needed, while parallelism is how many can run at once.

### Common Mistakes to Avoid
- Forgetting to set the namespace when creating the job
- Incorrectly formatting the command and args in the container spec
- Not setting the correct restart policy (should be `Never` for Jobs)
- Confusing `parallelism` with `completions`
- Forgetting to add labels to the pod template

## Additional Practice
1. Create a job that runs a Python script
2. Set up a job with resource limits
3. Configure a job with environment variables
4. Create a job that depends on the completion of another job

## Related Commands
```bash
# Delete the job (this will also clean up the pods)
kubectl delete job neb-new-job -n neptune

# Watch the job status
kubectl get jobs -n neptune -w

# Get detailed job information
kubectl describe job neb-new-job -n neptune

# View logs from all pods created by the job
kubectl logs -n neptune -l job-name=neb-new-job --all-containers=true
```

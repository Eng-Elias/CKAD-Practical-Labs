# CKAD Lab 30: Creating a Job with Specific Requirements

## Objective
Create a Kubernetes Job with specific parameters for completions, backoff limit, and restart policy, and understand the correct structure of a Job manifest.

## The Challenge
Create a Job with the following specifications:
-   **Name:** `foo-job`
-   **Image:** `busybox` (Note: The video uses `nginx`, but it lacks the `ping` command, so `busybox` is the correct choice).
-   **Command:** `ping -c 4 8.8.8.8` (ping Google's DNS server).
-   **Completions:** The Job must run to successful completion 9 times.
-   **Backoff Limit:** The Job should retry a failing pod up to 5 times before marking the Job as failed.
-   **Restart Policy:** The pods created by the Job should never be restarted (`Never`).

## Solution Strategy
The key to this lab is correctly placing the configuration fields. A `Job` manifest has two `spec` sections:
1.  **The Job `spec`:** This is the top-level `spec` that defines the Job's behavior (e.g., `completions`, `parallelism`, `backoffLimit`).
2.  **The Pod Template `spec`:** This is the nested `spec` (`spec.template.spec`) that defines the pods the Job will create (e.g., `containers`, `restartPolicy`).

### Step 1: Build the Job Manifest
Create a file named `my-job.yaml` and construct the manifest, paying close attention to where each field goes.

**`my-job.yaml`:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: foo-job
spec:
  # --- Job-Specific Settings ---
  completions: 9
  backoffLimit: 5

  # --- Pod Template: Blueprint for the Pods ---
  template:
    spec:
      # --- Pod-Specific Settings ---
      containers:
      - name: foo-container
        image: busybox
        command: ["ping", "-c", "4", "8.8.8.8"]
      
      # This policy applies to the pods created by the Job
      restartPolicy: Never
```

**Explanation of Fields:**
-   `spec.completions: 9`: Tells the Job controller to ensure 9 pods run to successful completion.
-   `spec.backoffLimit: 5`: If a pod fails, Kubernetes will try to replace it up to 5 times. After the 6th failure (1 initial + 5 retries), the Job itself will be marked as failed.
-   `spec.template.spec.restartPolicy: Never`: This is a requirement for Jobs. It means the kubelet will not restart a container if it fails. Instead, the Job controller will create a new pod to replace the failed one.

### Step 2: Apply and Verify the Job
Apply the manifest and use `kubectl` to monitor the Job's progress.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f my-job.yaml

# Watch the Job's status
kubectl get jobs --watch
# You will see the COMPLETIONS count increase from 0/9 to 9/9.

# Check the pods created by the Job
kubectl get pods
# You will see multiple pods being created until 9 have the 'Completed' status.

# Describe the job to see its configuration
kubectl describe job foo-job
```

## Exam Tip
Remember the hierarchy: `Job -> Job Spec -> Pod Template -> Pod Spec`. Fields that control the Job's overall execution (`completions`, `parallelism`, `backoffLimit`) go in the top-level `spec`. Fields that define the pods themselves (`containers`, `volumes`, `restartPolicy`) go in the nested `spec` under `template`. Using `kubectl explain job.spec` and `kubectl explain job.spec.template.spec` can quickly clarify where a field belongs.

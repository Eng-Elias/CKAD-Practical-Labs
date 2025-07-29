# CKAD Lab 56 (Part 1): Creating a CronJob

## Objective
Create a `CronJob` with a specific set of execution controls, such as parallelism, completion count, and failure backoff limits. This lab demonstrates how to use `kubectl explain` to find the correct fields within the `CronJob` spec.

## The Challenge
Create a `CronJob` named `foo-cron` with the following requirements:
-   The container should be named `foo-container` and use the `busybox` image.
-   It must not run in parallel (only one job at a time).
-   It must complete successfully `2` times.
-   It should give up after `15` failed attempts (`backoffLimit`).
-   The job should be terminated if it runs longer than `15` seconds.

## Part 1 Solution: Building the Manifest with `kubectl explain`

### Step 1: Start with a Base Manifest
As always, the fastest way to start is by finding a `CronJob` example in the official Kubernetes documentation and adapting it.

### Step 2: Use `kubectl explain` for Job-Specific Settings
For a `CronJob`, the specifications for the `Job` it creates are nested under `spec.jobTemplate.spec`. This is the path we need to explore.

**The Discovery Path:**
`kubectl explain cronjob.spec.jobTemplate.spec`

Running this command reveals the fields we need to control the job's execution:
-   `parallelism`: The desired number of concurrently running pods. We'll set this to `1` to meet the non-parallel requirement (or `0` as mentioned in the video, though `1` is more explicit for non-parallel execution).
-   `completions`: The desired number of successfully finished pods. We'll set this to `2`.
-   `backoffLimit`: The number of retries before considering a Job as failed. We'll set this to `15`.

### The `cronjob.yaml` (In Progress)
Based on the initial discovery, we can build the following manifest. Note how the execution control fields are placed under `spec.jobTemplate.spec`.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: foo-cron
spec:
  schedule: "*/1 * * * *" # Runs every minute for testing
  jobTemplate:
    spec:
      # --- Job Execution Controls ---
      parallelism: 1
      completions: 2
      backoffLimit: 15
      # -----------------------------
      template:
        spec:
          containers:
          - name: foo-container
            image: busybox
            command: ["/bin/sh", "-c", "echo hello; sleep 5; echo world"]
          restartPolicy: OnFailure
```

## Next Steps
The final requirement is to set a deadline for the job's execution. Part 2 will cover finding the `activeDeadlineSeconds` field and applying the final manifest.

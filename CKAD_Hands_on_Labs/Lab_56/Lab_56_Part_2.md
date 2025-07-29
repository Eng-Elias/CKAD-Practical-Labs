# CKAD Lab 56 (Part 2): Completing the CronJob and Verification

## Objective
Finalize the `CronJob` manifest by adding the `activeDeadlineSeconds` field to ensure the job terminates if it runs too long. Apply and verify the completed resource.

## Part 2 Solution: Adding the Final Field

### Step 1: Find the Deadline Field
Continuing with `kubectl explain cronjob.spec.jobTemplate.spec`, we can search for a field related to a deadline or timeout.

**Command:**
`kubectl explain cronjob.spec.jobTemplate.spec`

Looking through the output reveals `activeDeadlineSeconds`.
-   `activeDeadlineSeconds`: Specifies the duration in seconds that the Job may be active before the system tries to terminate it. This is a hard deadline.

### The Final `cronjob.yaml`
We add this final field to the `jobTemplate.spec` to complete the manifest.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: foo-cron
spec:
  schedule: "*/1 * * * *" # Runs every minute
  jobTemplate:
    spec:
      # The job will be terminated if it runs longer than 15 seconds
      activeDeadlineSeconds: 15
      # --- Other Job Execution Controls ---
      parallelism: 1
      completions: 2
      backoffLimit: 15
      # -----------------------------------
      template:
        spec:
          containers:
          - name: foo-container
            image: busybox
            # This command will succeed, but if it were to hang,
            # activeDeadlineSeconds would kill it.
            command: ["/bin/sh", "-c", "echo 'Hello from the cron job'"]
          restartPolicy: OnFailure
```

### Step 2: Apply and Verify
Apply the final manifest and use `kubectl get` and `kubectl describe` to verify that all the settings have been correctly applied.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f cronjob.yaml

# Check that the CronJob was created
kubectl get cronjob foo-cron

# Describe the CronJob to see all the configured details
kubectl describe cronjob foo-cron
```

**Expected `describe` Output Snippet:**
```
...
Schedule:                       */1 * * * *
Concurrency Policy:             Allow
Suspend:                        False
Successful Jobs History Limit:  3
Failed Jobs History Limit:      1
Starting Deadline Seconds:      <unset>
Selector:                       <unset>
Parallelism:                    1
Completions:                    2
Backoff Limit:                  15
Active Deadline Seconds:        15s
Pod Template:
...
```

## Exam Tip
`CronJobs` have two `spec` sections. The top-level `spec` is for the `CronJob` controller itself (e.g., `schedule`, `jobTemplate`). The nested `spec` inside `jobTemplate` is the template for the `Job` resources it will create. Fields that control the execution of a single job run, like `completions`, `parallelism`, `backoffLimit`, and `activeDeadlineSeconds`, always belong in the nested `jobTemplate.spec`.

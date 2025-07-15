# CKAD Lab 25 (Part 2): Correcting the CronJob with `kubectl explain`

## Objective
Finalize the CronJob manifest from Part 1 by adding settings for `completions`, `backoffLimit`, and `activeDeadlineSeconds`. Most importantly, this lab corrects the placement of all job-specific settings within the manifest.

## Recap of the Challenge
Create a CronJob with the following requirements:
-   Name: `my-cronjob`
-   Schedule: Runs every five minutes.
-   Image: `busybox` with a command.
-   Concurrency: Non-parallel (`Forbid`).
-   Completions: Should only complete once (`1`).
-   Backoff Limit: Retry a failed job up to 4 times.
-   Active Deadline: A job should not run for more than 30 seconds.

## The Core Concept: The Job Template
A `CronJob` is not a job itself; it is a **controller that creates Job objects** on a schedule. Therefore, all the settings that define how the actual `Job` should behave (`parallelism`, `completions`, `backoffLimit`, etc.) must be placed inside the **`jobTemplate`**. 

The `jobTemplate` is literally a blueprint for the Jobs that the CronJob will create.

### Finding the Correct Paths with `kubectl explain`
The mistake in the video was placing job settings directly under `spec`. Let's use `kubectl explain --recursive` correctly to find the true path for a field like `backoffLimit`.

**Command:**
```bash
kubectl explain cronjob --recursive | grep -i "backoffLimit"
```

**Correct Interpretation of the Output:**
When you look at the output, you must trace the indentation to find the full path:
```
KIND:     CronJob
VERSION:  batch/v1

FIELD:    spec <Object>

  FIELD:    jobTemplate <Object>         <-- Start of the Job Template

    FIELD:    spec <Object>              <-- The Job's own spec

      FIELD:    backoffLimit <integer>   <-- HERE IT IS!
```
This clearly shows the correct path is `spec.jobTemplate.spec.backoffLimit`.

## Assembling the Final, Correct Manifest
Now we can build the final `cronjob.yaml` with all fields in their correct locations.

**Final `cronjob.yaml`:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  # --- CronJob-Specific Settings ---
  schedule: "*/5 * * * *"
  concurrencyPolicy: Forbid
  
  # --- Template for the Jobs to be Created ---
  jobTemplate:
    spec:
      # --- Job-Specific Settings ---
      completions: 1
      backoffLimit: 4
      activeDeadlineSeconds: 30
      
      # --- Pod Template within the Job Template ---
      template:
        spec:
          containers:
          - name: my-cronjob-container
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo "Hello from CronJob"
            resources: {}
          restartPolicy: OnFailure
```

## Apply and Verify

**Commands:**
```bash
# Apply the correct manifest
kubectl apply -f cronjob.yaml

# Describe the cronjob to see the settings
kubectl describe cronjob my-cronjob
```
When you describe the CronJob, you will see all the settings correctly configured under the `Job Template` section.

## Exam Tip
Misunderstanding the nested structure of Kubernetes objects is a common source of errors. The `CronJob` -> `JobTemplate` -> `PodTemplate` structure is a prime example. Always use `kubectl explain --recursive` and pay close attention to the indentation to determine the correct path for a field. This will save you from frustrating validation errors during the exam.

# CKAD Lab 25 (Part 1): Building a CronJob with `kubectl explain`

## Objective
Learn how to use the `kubectl explain` command to discover the fields needed to build a complex Kubernetes manifest, such as a CronJob, without relying on external documentation. This lab also highlights a common pitfall and how to correct it.

## The Challenge
Create a CronJob with the following specifications:
-   Name: `my-cronjob`
-   Schedule: Runs every five minutes.
-   Image: `busybox`
-   Command: `echo "Hello from CronJob"`
-   Concurrency Policy: Must not run jobs in parallel (`Non-parallel`).

## Solution Strategy: Using `kubectl explain`

For complex objects like CronJobs, `kubectl explain` is faster than searching the web. It gives you direct access to the object's schema.

### Step 1: Generate the Base CronJob Manifest
First, generate a basic CronJob YAML file. This gives us a valid starting structure.

**Command:**
```bash
kubectl create cronjob my-cronjob --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > cronjob.yaml
```

### Step 2: Basic Modifications (Name and Schedule)
Edit `cronjob.yaml` to meet the first two requirements.

-   Change `schedule` from `"*/1 * * * *"` to `"*/5 * * * *"`.
-   Add a simple `args` to the container for the echo command.

**Initial `cronjob.yaml`:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - image: busybox
            name: my-cronjob
            args:
            - /bin/sh
            - -c
            - date; echo "Hello from CronJob"
            resources: {}
          restartPolicy: OnFailure
```

### Step 3: Finding the "Non-Parallel" Setting

The requirement for the CronJob to be "non-parallel" is ambiguous. It could mean several things. We'll use `kubectl explain` to find relevant fields.

**Command:**
```bash
# Recursively search the CronJob schema for fields related to concurrency or parallelism
kubectl explain cronjob --recursive | grep -i "parallel"
```

This search reveals two potentially relevant fields:
-   `concurrencyPolicy`
-   `parallelism`

Let's investigate `concurrencyPolicy` first.

```bash
kubectl explain cronjob.spec.concurrencyPolicy
```

The output shows three options: `Allow`, `Forbid`, and `Replace`. `Forbid` prevents concurrent runs, which perfectly matches our "non-parallel" requirement.

### The Common Mistake (and how to avoid it)

In the video, the user tries to set `parallelism`. Let's analyze that path. The `grep` command shows that `parallelism` exists, but where?

If you look closely at the output of `kubectl explain cronjob --recursive`, you will see the full path:
```
... 
spec
  jobTemplate
    spec
      parallelism    <-- HERE IT IS!
...
```

The `parallelism` field does not belong directly under `spec`. It belongs to the **Job template's spec**, which is nested inside the CronJob spec: `spec.jobTemplate.spec.parallelism`.

Placing `parallelism: 0` directly under `spec` will cause an error because the CronJob object itself has no such field.

### Correctly Setting the Concurrency Policy

For this lab, the most direct way to achieve the "non-parallel" goal is using `concurrencyPolicy`.

**Final `cronjob.yaml`:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"
  # This policy prevents new jobs from running if the previous one hasn't finished
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - image: busybox
            name: my-cronjob
            args:
            - /bin/sh
            - -c
            - date; echo "Hello from CronJob"
            resources: {}
          restartPolicy: OnFailure
```

This lab will be continued in Part 2.

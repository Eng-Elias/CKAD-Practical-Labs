# 6.6 Managing CronJobs

## Introduction to CronJobs

CronJobs in Kubernetes are used to schedule Jobs to run at specific times or intervals, similar to the Unix `cron` utility. They are ideal for periodic and recurring tasks like backups, reports, and maintenance operations.

### Key Features
- **Time-based Scheduling**: Uses standard cron syntax
- **Job History**: Keeps track of successful and failed jobs
- **Concurrency Policies**: Control how concurrent executions are handled
- **Suspension**: Temporarily disable scheduling without deleting the CronJob

## Basic CronJob Examples

### Simple CronJob
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"  # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo "Hello from Kubernetes"
          restartPolicy: OnFailure
```

### CronJob with Environment Variables
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: env-printer
spec:
  schedule: "0 * * * *"  # At the start of every hour
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: env-printer
            image: busybox:1.28
            env:
            - name: ENVIRONMENT
              value: "production"
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            command: ["sh", "-c", "echo Running on $NODE_NAME in $ENVIRONMENT"]
          restartPolicy: OnFailure
```

## Advanced CronJob Features

### Concurrency Policies
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: concurrency-demo
spec:
  schedule: "*/5 * * * *"
  concurrencyPolicy: Forbid  # Options: Allow, Forbid, Replace
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: long-running
            image: busybox:1.28
            command: ["sh", "-c", "sleep 300"]
          restartPolicy: OnFailure
```

### Starting Deadline
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: with-deadline
spec:
  schedule: "0 * * * *"
  startingDeadlineSeconds: 60  # Start job if missed within 60s of scheduled time
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: deadline-demo
            image: busybox:1.28
            command: ["sh", "-c", "date; echo Starting job"]
          restartPolicy: OnFailure
```

### Successful Jobs History Limit
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-limit-demo
spec:
  schedule: "*/2 * * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: history-demo
            image: busybox:1.28
            command: ["sh", "-c", "date; echo This is a demo"]
          restartPolicy: OnFailure
```

## Real-world Examples

### Database Backup
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"  # 2 AM every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:13
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
            command: 
            - /bin/sh
            - -c
            - |
              pg_dump -h postgres -U postgres mydb > /backup/mydb-$(date +%Y%m%d).sql
              gzip /backup/mydb-$(date +%Y%m%d).sql
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          restartPolicy: OnFailure
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: backup-pvc
```

### Periodic Report Generation
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekly-report
spec:
  schedule: "0 0 * * 0"  # Midnight on Sunday
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: report-generator
            image: myapp/report-generator:1.0.0
            env:
            - name: REPORT_PERIOD
              value: "weekly"
            - name: SMTP_SERVER
              value: "smtp.example.com"
          restartPolicy: OnFailure
```

## Managing CronJobs

### Common Commands
```bash
# List all CronJobs
kubectl get cronjobs

# View CronJob details
kubectl describe cronjob/my-cronjob

# View jobs created by a CronJob
kubectl get jobs --watch

# View logs of a specific job
kubectl logs job/my-cronjob-1234567890

# Manually trigger a CronJob
kubectl create job --from=cronjob/my-cronjob manual-$(date +%s)

# Suspend a CronJob
kubectl patch cronjob my-cronjob -p '{"spec" : {"suspend" : true }}'

# Resume a CronJob
kubectl patch cronjob my-cronjob -p '{"spec" : {"suspend" : false }}'

# Delete a CronJob and its jobs
kubectl delete cronjob/my-cronjob
```

## Troubleshooting

### Common Issues

#### CronJob Not Creating Jobs
```bash
# Check CronJob status
kubectl describe cronjob/my-cronjob

# Check controller manager logs
kubectl logs -n kube-system -l component=kube-controller-manager | grep -i cronjob

# Check for schedule format issues
kubectl get cronjob my-cronjob -o yaml | grep schedule
```

#### Jobs Not Completing
```bash
# Check job status
kubectl describe job/my-cronjob-1234567890

# View pod logs
kubectl logs job/my-cronjob-1234567890

# Check for resource constraints
kubectl describe nodes | grep -A 10 "Allocated resources"
```

### Debugging Tips

#### Check Events
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

#### View Controller Logs
```bash
kubectl logs -n kube-system -l component=kube-controller-manager | grep -i cronjob
```

## Best Practices

1. **Naming Conventions**
   - Use descriptive names that indicate the purpose and schedule
   - Example: `daily-backup`, `monthly-report`

2. **Resource Management**
   - Always set resource requests and limits
   - Consider using `startingDeadlineSeconds` for critical jobs

3. **Error Handling**
   - Implement proper error handling in your jobs
   - Set appropriate `restartPolicy` and `backoffLimit`

4. **Cleanup**
   - Set `successfulJobsHistoryLimit` and `failedJobsHistoryLimit`
   - Regularly clean up old jobs

5. **Monitoring**
   - Monitor CronJob status and job history
   - Set up alerts for failed jobs

## Time Zone Considerations

By default, CronJobs use the time zone of the Kubernetes control plane. To use a specific time zone:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: timezone-demo
spec:
  schedule: "0 8 * * *"  # 8 AM in the specified time zone
  timeZone: "America/New_York"  # Requires Kubernetes 1.24+
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: timezone
            image: busybox:1.28
            command: ["sh", "-c", "date; echo 'Running in America/New_York time zone'"]
          restartPolicy: OnFailure
```

## Next Steps
- Learn about Kubernetes Jobs for one-time tasks
- Explore Kubernetes Operators for complex scheduling
- Understand Pod Lifecycle and Hooks
- Study Kubernetes API for advanced automation

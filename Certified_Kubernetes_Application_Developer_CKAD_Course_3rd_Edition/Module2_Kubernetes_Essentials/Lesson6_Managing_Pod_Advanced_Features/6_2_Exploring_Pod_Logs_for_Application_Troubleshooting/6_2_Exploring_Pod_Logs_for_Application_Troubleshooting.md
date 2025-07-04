# 6.2 Exploring Pod Logs for Application Troubleshooting

## Introduction to Pod Logs

In cloud-native environments, applications don't have a traditional console output. Instead, they write logs to standard output (stdout) and standard error (stderr), which Kubernetes collects and makes available through the `kubectl logs` command.

### Key Concepts
- **Log Collection**: Kubernetes automatically collects logs from container stdout/stderr
- **Log Retention**: Logs are retained as long as the pod exists
- **Multi-container Pods**: Each container's logs are stored separately
- **Log Rotation**: Managed by the container runtime (e.g., Docker)

## Basic Log Commands

### Viewing Logs
```bash
# Basic log viewing
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>

# View logs from the last N lines (default 10)
kubectl logs --tail=20 <pod-name>

# View logs since a specific duration (e.g., 5m, 1h)
kubectl logs --since=1h <pod-name>
```

### Multi-container Pods
```bash
# Specify container name
kubectl logs <pod-name> -c <container-name>

# View logs from all containers
kubectl logs <pod-name> --all-containers

# View previous instance logs if container restarted
kubectl logs -p <pod-name>
```

## Troubleshooting with Logs

### Common Issues and Solutions

#### 1. CrashLoopBackOff
```bash
# Check why the pod is crashing
kubectl logs <pod-name> --previous

# Check pod status and events
kubectl describe pod <pod-name>
```

#### 2. Container Not Starting
```bash
# Check init container logs
kubectl logs <pod-name> -c <init-container-name>

# Check pod events
kubectl get events --sort-by=.metadata.creationTimestamp
```

#### 3. Application Errors
```bash
# Follow logs with timestamps
kubectl logs -f <pod-name> --timestamps

# Filter logs for errors
kubectl logs <pod-name> | grep -i error
```

## Real-world Example: Debugging a Database Pod

### 1. Check Pod Status
```bash
kubectl get pods
# Output: mydb-xyz123   0/1     CrashLoopBackOff   3          2m
```

### 2. View Pod Details
```bash
kubectl describe pod mydb-xyz123
# Look for Events section and Container Status
```

### 3. Check Logs
```bash
kubectl logs mydb-xyz123
# Output might show: "Database is uninitialized and password option is not specified"
```

### 4. Fix the Issue
```bash
# Delete the failing pod (if part of a controller, it will be recreated)
kubectl delete pod mydb-xyz123

# Create a new pod with proper environment variables
kubectl run mydb --image=mariadb --env="MYSQL_ROOT_PASSWORD=password"
```

## Advanced Logging Techniques

### Log Formatting
```bash
# Show timestamps with logs
kubectl logs --timestamps <pod-name>

# Add line numbers
kubectl logs --tail=50 <pod-name> | nl

# Colorize output (requires 'grc')
kubectl logs <pod-name> | grep --color -E '^|error|fail|warn|exception'
```

### Logging from Multiple Pods
```bash
# Get logs from pods with a specific label
kubectl logs -l app=myapp --all-containers

# Get logs from a specific container in multiple pods
kubectl logs -l app=myapp -c mycontainer

# Get logs from all pods in a namespace
kubectl logs --all-containers --namespace=my-namespace --selector=app=myapp
```

### Log Rotation and Retention
```bash
# Configure log rotation in container runtime (Docker example)
# Add to /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

## Best Practices for Application Logging

1. **Log to stdout/stderr**
   - Don't write to log files inside containers
   - Let Kubernetes handle log collection

2. **Use Structured Logging**
   - Output logs in JSON format
   - Include timestamps and log levels

3. **Meaningful Messages**
   - Include context in log messages
   - Use consistent log levels (INFO, DEBUG, ERROR, etc.)

4. **Sensitive Information**
   - Never log sensitive data (passwords, tokens, PII)
   - Use Kubernetes secrets for sensitive configuration

5. **Log Volume**
   - Be mindful of log volume
   - Use appropriate log levels to control verbosity

## Example: Logging in Different Programming Languages

### Python
```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    stream=sys.stdout
)
logger = logging.getLogger(__name__)

logger.info("Application started")
try:
    # Your application code
    logger.debug("Debug information")
except Exception as e:
    logger.error(f"An error occurred: {str(e)}", exc_info=True)
```

### Node.js
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [new winston.transports.Console()]
});

logger.info('Application started');
```

## Monitoring and Alerting

### Setting Up Log-based Alerts
```bash
# Watch for error patterns
kubectl logs -f deployment/myapp | grep --line-buffered -i -E 'error|exception|fail' | \
while read line; do
  echo "[ERROR DETECTED] $line"
  # Add alerting logic here
  # e.g., send email, Slack notification, etc.
done
```

### Log Aggregation Solutions
1. **EFK Stack**
   - Elasticsearch
   - Fluentd/Fluent Bit
   - Kibana

2. **Loki Stack**
   - Loki
   - Promtail
   - Grafana

3. **Cloud Solutions**
   - AWS CloudWatch
   - Google Cloud Logging
   - Azure Monitor

## Next Steps
- Learn about Kubernetes Events for better troubleshooting
- Explore log aggregation solutions
- Understand pod lifecycle and probes
- Study monitoring and observability in Kubernetes

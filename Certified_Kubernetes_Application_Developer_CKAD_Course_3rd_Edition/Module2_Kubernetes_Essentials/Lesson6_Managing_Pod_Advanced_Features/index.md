# Lesson 6: Managing Pods - Advanced Features

## Overview
This lesson dives into advanced Pod management techniques in Kubernetes. You'll learn how to troubleshoot Pods, manage resources, implement security contexts, and work with Jobs and CronJobs. These skills are essential for running production workloads in Kubernetes.

## Learning Objectives
- Debug and troubleshoot Pods effectively
- Use port forwarding for local access to Pods
- Implement security contexts for Pods and containers
- Manage batch workloads with Jobs and CronJobs
- Configure resource limitations and quotas
- Clean up resources efficiently

## Lessons

### [6.1 Exploring Pod State with kubectl describe](6_1_Exploring_Pod_State_with_kubectl_describe/6_1_Exploring_Pod_State_with_kubectl_describe.md)
- Understanding Pod status and conditions
- Using `kubectl describe` for debugging
- Common Pod states and their meanings
- Events and their significance

### [6.2 Exploring Pod Logs for Application Troubleshooting](6_2_Exploring_Pod_Logs_for_Application_Troubleshooting/6_2_Exploring_Pod_Logs_for_Application_Troubleshooting.md)
- Accessing container logs
- Log aggregation and management
- Debugging application issues
- Log rotation and retention

### [6.3 Using Port Forwarding to Access Pods](6_3_Using_Port_Forwarding_to_Access_Pods/6_3_Using_Port_Forwarding_to_Access_Pods.md)
- Port forwarding concepts
- Local port forwarding to Pods
- Debugging with port forwarding
- Security considerations

### [6.4 Understanding and Configuring SecurityContext](6_4_Understanding_and_Configuring_SecurityContext/6_4_Understanding_and_Configuring_SecurityContext.md)
- Pod and container security contexts
- Running as non-root users
- Capabilities and privileges
- Security policies and pod security standards

### [6.5 Managing Jobs](6_5_Managing_Jobs/6_5_Managing_Jobs.md)
- Understanding Kubernetes Jobs
- Creating and managing Jobs
- Job patterns and use cases
- Handling Job failures and retries

### [6.6 Managing CronJobs](6_6_Managing_CronJobs/6_6_Managing_CronJobs.md)
- Introduction to CronJobs
- Cron syntax and scheduling
- Managing CronJob history
- Monitoring and alerts

### [6.7 Managing Resource Limitations and Quota](6_7_Managing_Resource_Limitations_and_Quota/6_7_Managing_Resource_Limitations_and_Quota.md)
- Resource requests and limits
- Quality of Service (QoS) classes
- Resource quotas
- Limit ranges

### [6.8 Cleaning Up Resources](6_8_Cleaning_Up_Resources/6_8_Cleaning_Up_Resources.md)
- Managing resource cleanup
- Using labels and selectors
- Cascading deletes
- Namespace cleanup

## Hands-on Exercises
1. Debug a failing Pod using `kubectl describe` and logs
2. Set up port forwarding to access a Pod
3. Configure security contexts for a Pod
4. Create and manage a Job and CronJob
5. Set resource limits and quotas
6. Clean up resources using labels and selectors

## Advanced Patterns
- **Job Patterns**: Parallel Jobs, Work Queues, Indexed Jobs
- **Security Patterns**: Non-root containers, read-only root filesystems
- **Resource Management**: Burstable vs. Guaranteed Pods
- **Cleanup Strategies**: TTL controllers, owner references

## Best Practices
- Always set resource requests and limits
- Implement proper security contexts
- Use Jobs for batch processing
- Monitor resource usage
- Clean up unused resources
- Document your configurations

## Troubleshooting
- Pod scheduling failures
- Container runtime issues
- Resource starvation
- Permission denied errors
- Job completion and failures

## Next Steps
After mastering advanced Pod management, you'll be ready to learn about [Deployments in Lesson 7](../Lesson7_Managing_Deployments/index.md), where you'll manage declarative updates for Pods and ReplicaSets.
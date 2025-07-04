# Lesson 14: Troubleshooting Kubernetes

## Overview
This lesson provides a systematic approach to troubleshooting applications and infrastructure in Kubernetes. You'll learn how to diagnose and resolve common issues, from application failures to cluster-level problems, using native Kubernetes tools and best practices.

## Learning Objectives
- Develop a structured approach to troubleshooting
- Diagnose and resolve application failures
- Troubleshoot networking and access issues
- Monitor and analyze cluster events
- Implement effective health checks
- Secure and debug authentication problems

## Topics

### [14.1 Determining a Troubleshooting Strategy](14.1_Determining_a_Troubleshooting_Strategy/14.1_Determining_a_Troubleshooting_Strategy.md)
- The troubleshooting methodology
- Gathering relevant information
- Common failure patterns
- Documenting the process

### [14.2 Analyzing Failing Applications](14.2_Analyzing_Failing_Applications/14.2_Analyzing_Failing_Applications.md)
- Pod lifecycle and states
- Container logs analysis
- Resource constraints and OOM issues
- Application dependency problems

### [14.3 Analyzing Pod Access Problems](14.3_Analyzing_Pod_Access_Problems/14.3_Analyzing_Pod_Access_Problems.md)
- Network policies and connectivity
- Service discovery issues
- DNS resolution problems
- Ingress and service configuration

### [14.4 Monitoring Cluster Event Logs](14.4_Monitoring_Cluster_Event_Logs/14.4_Monitoring_Cluster_Event_Logs.md)
- Kubernetes event stream
- Metrics and monitoring tools
- Log aggregation and analysis
- Setting up alerts

### [14.5 Troubleshooting Authentication Problems](14.5_Troubleshooting_Authentication_Problems/14.5_Troubleshooting_Authentication_Problems.md)
- Authentication mechanisms
- RBAC configuration issues
- Service account problems
- Kubeconfig and context issues

### [14.6 Using Probes](14.6_Using_Probes/14.6_Using_Probes.md)
- Liveness, readiness, and startup probes
- Configuring effective health checks
- Common probe patterns
- Debugging probe failures

## Hands-on Exercises
1. Debug a crashing application pod
2. Troubleshoot a service connectivity issue
3. Analyze and fix RBAC permission problems
4. Configure and test liveness/readiness probes
5. Monitor and interpret cluster events
6. Resolve common authentication issues

## Troubleshooting Workflow
1. **Observe**: Gather information about the issue
2. **Reproduce**: Consistently replicate the problem
3. **Isolate**: Narrow down the root cause
4. **Resolve**: Implement and test the fix
5. **Document**: Record the solution for future reference

## Best Practices
- Always check pod status and events first
- Use `kubectl describe` for detailed resource information
- Implement structured logging in applications
- Set appropriate resource requests and limits
- Use namespaces to isolate environments
- Regularly review and update RBAC policies

## Common Tools and Commands
- `kubectl get events --all-namespaces --sort-by='.metadata.creationTimestamp'
- `kubectl logs -f <pod-name> -c <container-name>`
- `kubectl describe pod <pod-name>`
- `kubectl exec -it <pod-name> -- /bin/sh`
- `kubectl port-forward svc/<service-name> <local-port>:<service-port>`
- `kubectl auth can-i <verb> <resource>`

## Next Steps
After mastering troubleshooting techniques, you'll be ready to explore [Module 5: Sample Exam](../../Module5_Sample_Exam/index.md) to learn about implementing sophisticated deployment strategies and managing stateful applications.
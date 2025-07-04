# Lesson 5: Managing Pods - Basic Features

## Overview
This lesson introduces the fundamental concepts of working with Pods in Kubernetes. You'll learn how to create, manage, and configure Pods, which are the smallest deployable units in the Kubernetes ecosystem. The lesson covers YAML manifests, multi-container Pods, init containers, and namespaces.

## Learning Objectives
- Understand the purpose and lifecycle of Pods
- Create and manage Pods using YAML manifests
- Work with multi-container Pods
- Implement init containers for initialization tasks
- Organize resources using namespaces
- Generate YAML files using imperative commands

## Lessons

### [5.1 Understanding Pods](5_1_Understanding_Pods/5_1_Understanding_Pods.md)
- What is a Pod?
- Pod lifecycle and states
- Single-container vs. multi-container Pods
- Pod networking and storage

### [5.2 Understanding YAML](5_2_Understanding_YAML/5_2_Understanding_YAML.md)
- YAML syntax and structure
- Kubernetes YAML manifest structure
- Required and optional fields
- Common YAML patterns in Kubernetes

### [5.3 Generating YAML Files](5_3_Generating_YAML_Files/5_3_Generating_YAML_Files.md)
- Using `kubectl run` with `--dry-run=client`
- Exporting existing resource configurations
- YAML validation and linting
- Best practices for YAML management

### [5.4 Understanding and Configuring Multi-Container Pods](5_4_Understanding_and_Configuring_Multi-Container_Pods/5_4_Understanding_and_Configuring_Multi-Container_Pods.md)
- Use cases for multi-container Pods
- Container communication within Pods
- Shared volumes in multi-container Pods
- Sidecar and adapter patterns

### [5.5 Working with Init Containers](5_5_Working_with_Init_Containers/5_5_Working_with_Init_Containers.md)
- Init container concepts
- Common init container patterns
- Resource requirements and limitations
- Debugging init containers

### [5.6 Using Namespaces](5_6_Using_Namespaces/5_6_Using_Namespaces.md)
- Namespace concepts and use cases
- Creating and managing namespaces
- Resource quotas and limits
- Cross-namespace communication

## Hands-on Exercises
1. Create a simple Pod using YAML
2. Generate YAML files using `kubectl`
3. Deploy a multi-container Pod
4. Implement an init container for setup tasks
5. Organize resources using namespaces
6. Clean up resources

## Common Pod Patterns
- **Sidecar**: Extends and enhances the main container
- **Adapter**: Standardizes and normalizes output
- **Ambassador**: Proxies local connections to the network
- **Init Containers**: Run to completion before app containers start

## Best Practices
- Always define resource requests and limits
- Use meaningful labels and selectors
- Implement proper health checks
- Use namespaces for environment separation
- Follow the principle of least privilege
- Document your Pod specifications

## Troubleshooting
- Pod stays in Pending state
- Container creation errors
- Image pull failures
- Resource constraints
- Network connectivity issues

## Next Steps
After mastering basic Pod management, proceed to [Lesson 6: Managing Pods - Advanced Features](../Lesson6_Managing_Pod_Advanced_Features/index.md) to learn about advanced Pod management techniques, including security contexts, resource management, and troubleshooting.
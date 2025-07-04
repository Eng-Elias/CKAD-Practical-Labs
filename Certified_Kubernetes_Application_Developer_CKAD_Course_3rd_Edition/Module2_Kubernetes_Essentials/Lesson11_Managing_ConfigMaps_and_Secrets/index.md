# Lesson 11: Managing ConfigMaps and Secrets

## Overview
This lesson covers how to manage configuration data and sensitive information in Kubernetes using ConfigMaps and Secrets. You'll learn best practices for decoupling configuration from container images and securely managing sensitive data.

## Learning Objectives
- Understand the importance of decoupling configuration
- Create and manage ConfigMaps
- Work with environment variables and configuration files
- Implement Kubernetes Secrets securely
- Configure container registry authentication
- Follow security best practices

## Lessons

### [11.1 Providing Variables to Kubernetes Applications](11_1_Providing_Variables_to_Kubernetes_Applications/11_1_Providing_Variables_to_Kubernetes_Applications.md)
- Configuration management approaches
- Environment variables in Kubernetes
- Command-line arguments
- Best practices for configuration

### [11.2 Understanding Why Decoupling Is Important](11_2_Understanding_Why_Decoupling_Is_Important/11_2_Understanding_Why_Decoupling_Is_Important.md)
- Configuration as code
- Environment parity
- Security considerations
- Deployment flexibility

### [11.3 Providing Variables with ConfigMaps](11_3_Providing_Variables_with_ConfigMaps/11_3_Providing_Variables_with_ConfigMaps.md)
- Creating ConfigMaps
- Using ConfigMaps as environment variables
- ConfigMap updates and versioning
- Resource consumption considerations

### [11.4 Providing Configuration Files Using ConfigMaps](11_4_Providing_Configuration_Files_Using_ConfigMaps/11_4_Providing_Configuration_Files_Using_ConfigMaps.md)
- Mounting ConfigMaps as volumes
- File-based configuration
- SubPath usage
- ConfigMap reloading

### [11.5 Understanding Secrets](11_5_Understanding_Secrets/11_5_Understanding_Secrets.md)
- What are Kubernetes Secrets?
- Secret types
- Secret management best practices
- Security considerations

### [11.6 Understanding How Kubernetes Uses Secrets](11_6_Understanding_How_Kubernetes_Uses_Secrets/11_6_Understanding_How_Kubernetes_Uses_Secrets.md)
- Secret storage and encryption
- Access control for Secrets
- Secret rotation strategies
- Monitoring Secret usage

### [11.7 Configuring Applications to Use Secrets](11_7_Configuring_Applications_to_Use_Secrets/11_7_Configuring_Applications_to_Use_Secrets.md)
- Mounting Secrets as files
- Using Secrets as environment variables
- Image pull Secrets
- Security context constraints

### [11.8 Configuring the Docker Registry Access Secret](11_8_Configuring_the_Docker_Registry_Access_Secret/11_8_Configuring_the_Docker_Registry_Access_Secret.md)
- Private registry authentication
- Creating docker-registry Secrets
- Using imagePullSecrets
- Managing registry credentials

## Hands-on Exercises
1. Create and use a ConfigMap for application settings
2. Mount a ConfigMap as a volume
3. Create and use a generic Secret
4. Configure a Pod to use a docker-registry Secret
5. Implement configuration updates without Pod restarts
6. Secure sensitive data using Secrets

## Best Practices
- Never store sensitive data in container images
- Use ConfigMaps for non-sensitive configuration
- Always use Secrets for sensitive data
- Implement proper RBAC for Secrets
- Rotate Secrets regularly
- Monitor Secret access
- Use external secret management when needed

## Common Issues and Solutions
- **Configuration Not Updating**: Check volume mount type and update strategy
- **Permission Denied**: Verify Pod security context and volume permissions
- **Secret Not Found**: Check namespace and access controls
- **Registry Authentication Fails**: Verify credentials and Secret configuration

## Next Steps
After mastering ConfigMaps and Secrets, you'll be well-prepared to explore more advanced Kubernetes topics in the next module, where you'll learn about networking, security, and advanced application deployment strategies [Module 3: Building and Exposing Scalable Applications](../../Module3_Building_and_Exposing_Scalable_Applications/index.md).
# Lesson 12: Using the Kubernetes API

## Overview
This lesson dives deep into the Kubernetes API, the fundamental interface for interacting with your Kubernetes cluster. You'll learn how to work with API resources programmatically, manage authentication and authorization, and extend Kubernetes functionality using custom resources.

## Learning Objectives
- Understand the Kubernetes API architecture
- Interact with the API using kubectl and direct HTTP calls
- Manage authentication and authorization
- Work with Custom Resource Definitions (CRDs)
- Implement Role-Based Access Control (RBAC)
- Configure and manage ServiceAccounts

## Topics

### [12.1 Understanding the Kubernetes API](12.1_Understanding_the_Kubernetes_API/12.1_Understanding_the_Kubernetes_API.md)
- API server architecture
- API groups and versions
- API discovery
- API deprecation policy

### [12.2 Using curl to Work with API Objects](12.2_Using_curl_to_Work_with_API_Objects/12.2_Using_curl_to_Work_with_API_Objects.md)
- Direct API access with curl
- Authentication methods
- Common API operations (GET, POST, PUT, PATCH, DELETE)
- Working with different output formats

### [12.3 Understanding API Descriptions and Deprecations](12.3_Understanding_API_Descriptions_and_Deprecations/12.3_Understanding_API_Descriptions_and_Deprecations.md)
- API versioning
- API deprecation policy
- API discovery mechanisms
- API documentation

### [12.4 Authentication and Authorization](12.4_Authentication_and_Authorization/12.4_Authentication_and_Authorization.md)
- Authentication methods
- Service accounts
- Webhook authentication
- Token review API

### [12.5 API Access and ServiceAccounts](12.5_API_Access_and_ServiceAccounts/12.5_API_Access_and_ServiceAccounts.md)
- ServiceAccount resources
- Automounting ServiceAccount tokens
- Managing ServiceAccount tokens
- Best practices for ServiceAccounts

### [12.6 Role Based Access Control (RBAC)](12.6_Role_Based_Access_Control_RBAC/12.6_Role_Based_Access_Control_RBAC.md)
- RBAC concepts
- Roles and ClusterRoles
- RoleBindings and ClusterRoleBindings
- Common RBAC patterns

### [12.7 Configuring a ServiceAccount](12.7_Configuring_a_ServiceAccount/12.7_Configuring_a_ServiceAccount.md)
- Creating and managing ServiceAccounts
- Image pull secrets
- Automounting API credentials
- Security best practices

## Hands-on Exercises
1. Interact with the API server using curl
2. Create and manage custom resources
3. Configure RBAC for a microservice
4. Set up ServiceAccounts for applications
5. Implement custom admission controllers
6. Monitor API server metrics

## Best Practices
- Use kubectl proxy for local development
- Implement the principle of least privilege
- Regularly audit RBAC configurations
- Use ServiceAccounts for pod authentication
- Monitor API server metrics
- Keep API clients up to date

## Common Commands
```bash
# Get API resources
kubectl api-resources

# Get API versions
kubectl api-versions

# Access API directly
kubectl get --raw /api/v1/namespaces/default/pods

# Create ServiceAccount
kubectl create serviceaccount myapp

# Create Role and RoleBinding
kubectl create role pod-reader --verb=get --verb=list --resource=pods
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:myapp
```

## Resources
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Kubernetes API Concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [ServiceAccounts Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

## Next Steps
After mastering the Kubernetes API, you'll be ready to explore [Module 4: Advanced CKAD Tasks](../../Module4_Advanced_CKAD_Tasks/index.md), where you'll learn about Helm, Kustomize, and advanced deployment strategies.
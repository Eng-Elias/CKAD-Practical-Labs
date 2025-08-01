# Lesson 13: Deploying Applications the DevOps Way

## Overview
This lesson covers modern application deployment strategies and tools in the Kubernetes ecosystem. You'll learn how to package, deploy, and manage applications using industry-standard DevOps practices and tools like Helm and Kustomize.

## Learning Objectives
- Package applications using Helm charts
- Manage Kubernetes manifests with Kustomize
- Implement advanced deployment strategies
- Work with Custom Resource Definitions (CRDs)
- Deploy stateful applications
- Automate application deployments

## Topics

### [13.1 Using the Helm Package Manager](../../Module3_Building_and_Exposing_Scalable_Applications/Lesson13_Deploying_Application_the_DevOps_Way/13_1_Using_the_Helm_Package_Manager/13_1_Using_the_Helm_Package_Manager.md)
- Helm architecture and components
- Installing and configuring Helm
- Working with charts and repositories
- Helm release management

### [13.2 Working with Helm Charts](../../Module3_Building_and_Exposing_Scalable_Applications/Lesson13_Deploying_Application_the_DevOps_Way/13_2_Working_with_Helm_Charts/13_2_Working_with_Helm_Charts.md)
- Chart structure and templates
- Values and value files
- Template functions and pipelines
- Chart dependencies
- Testing and verifying charts

### [13.3 Using Kustomize](../../Module3_Building_and_Exposing_Scalable_Applications/Lesson13_Deploying_Application_the_DevOps_Way/13_3_Using_Kustomize/13_3_Using_Kustomize.md)
- Kustomize overview and concepts
- Kustomization files
- Bases and overlays
- Common transformations
- Working with secrets and config maps

### [13.4 Implementing Blue-Green Deployments](13_4_Implementing_Blue-Green_Deployments/13_4_Implementing_Blue-Green_Deployments.md)
- Blue-green deployment concepts
- Implementing with Services and Deployments
- Traffic switching strategies
- Rollback procedures

### [13.5 Implementing Canary Deployments](13_5_Implementing_Canary_Deployments/13_5_Implementing_Canary_Deployments.md)
- Canary release patterns
- Traffic splitting with Service Meshes
- Metrics and monitoring
- Progressive delivery

### [13.6 Understanding Custom Resource Definitions](13_6_Understanding_Custom_Resource_Definitions/13_6_Understanding_Custom_Resource_Definitions.md)
- CRD concepts
- Defining custom resources
- Validation and defaulting
- Status subresource

### [13.7 Using Operators](13_7_Using_Operators/13_7_Using_Operators.md)
- Operator pattern
- Operator SDK
- Building a simple operator
- Best practices

### [13.8 Working with StatefulSets](13_8_Working_with_StatefulSets/13_8_Working_with_StatefulSets.md)
- StatefulSet concepts
- Pod identity and stable storage

## Hands-on Exercises
1. Create and package a Helm chart
2. Deploy an application using Kustomize
3. Set up a blue-green deployment
4. Implement a canary release
5. Create and use a Custom Resource Definition
6. Deploy a stateful application

## Best Practices
- Version your Helm charts
- Use semantic versioning for releases
- Implement proper rollback strategies
- Monitor deployment health
- Secure your deployments
- Document your deployment processes

## Common Commands
```bash
# Install a Helm chart
helm install my-release stable/nginx-ingress

# Create a new Helm chart
helm create mychart

# Apply Kustomize configuration
kubectl apply -k ./overlays/prod

# List Helm releases
helm list

# Rollback a Helm release
helm rollback my-release 1

# View Kubernetes resources with Kustomize
kubectl kustomize ./base
```

## Resources
- [Helm Documentation](https://helm.sh/docs/)
- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Operator SDK](https://sdk.operatorframework.io/)

## Next Steps
After mastering application deployment, you'll be ready to explore [Module 4: Advanced CKAD Tasks](../../Module4_Advanced_CKAD_Tasks/index.md), where you'll learn about troubleshooting, monitoring, and optimizing your Kubernetes applications.
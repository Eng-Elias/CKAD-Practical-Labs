# Lesson 4: Creating a Lab Environment

## Overview
This lesson guides you through setting up a local Kubernetes development environment using Minikube. You'll learn how to install and configure Minikube on different operating systems and run your first application in a local Kubernetes cluster.

## Learning Objectives
- Understand different Kubernetes deployment options
- Install and configure Minikube on your local machine
- Verify the Kubernetes cluster is working correctly
- Deploy and access your first application
- Troubleshoot common installation issues

## Lessons

### [4.1 Understanding Kubernetes Deployment Options](4_1_Understanding_Kubernetes_Deployment_Options/4_1_Understanding_Kubernetes_Deployment_Options.md)
- Local development options (Minikube, Docker Desktop, kind)
- Managed Kubernetes services (EKS, AKS, GKE)
- On-premises vs. cloud deployments
- Choosing the right deployment option

### [4.2 Using Minikube](4_2_Using_Minikube/4_2_Using_Minikube.md)
- Introduction to Minikube
- Minikube features and capabilities
- Starting and stopping the Minikube cluster
- Basic Minikube commands

### [4.3 Installing Minikube on Ubuntu](4_3_Installing_Minikube_on_Ubuntu/4_3_Installing_Minikube_on_Ubuntu.md)
- System requirements
- Installing dependencies
- Minikube installation steps
- Post-installation verification

### [4.4 Installing Minikube on Windows](4_4_Installing_Minikube_on_Windows/4_4_Installing_Minikube_on_Windows.md)
- Windows system requirements
- Enabling Hyper-V or VirtualBox
- Installation using Chocolatey
- Manual installation steps

### [4.5 Installing Minikube on macOS](4_5_Installing_Minikube_on_macOS/4_5_Installing_Minikube_on_macOS.md)
- macOS system requirements
- Installation using Homebrew
- Manual installation steps
- Configuring the hypervisor

### [4.6 Verifying Minikube is Working](4_6_Verifying_Minikube_is_Working/4_6_Verifying_Minikube_is_Working.md)
- Starting the Minikube cluster
- Verifying cluster status
- Accessing the Kubernetes dashboard
- Basic cluster operations

### [4.7 Running your First Application](4_7_Running_your_First_Application/4_7_Running_your_First_Application.md)
- Deploying a sample application
- Accessing the application
- Scaling the application
- Cleaning up resources

## Hands-on Exercises
1. Install Minikube on your preferred operating system
2. Start and verify the Minikube cluster
3. Deploy a sample application
4. Access the application using different methods
5. Scale and manage the application

## Common Issues and Solutions
- **Hypervisor Issues**: Ensure virtualization is enabled in BIOS
- **Network Problems**: Check proxy settings and network connectivity
- **Resource Constraints**: Allocate sufficient CPU and memory to Minikube
- **Port Conflicts**: Resolve any port conflicts with existing services

## Best Practices
- Always check system requirements before installation
- Keep Minikube and kubectl up to date
- Use version pinning for production-like environments
- Clean up unused resources when not in use
- Document your environment configuration

## Next Steps
After setting up your lab environment, you're ready to start working with Kubernetes pods in [Lesson 5: Managing Pods - Basic Features](../Lesson5_Managing_Pods_Basic_Features/index.md).
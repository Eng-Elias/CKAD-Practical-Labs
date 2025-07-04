# 13.1 Using the Helm Package Manager

## Introduction to Helm

Helm is a package manager for Kubernetes that streamlines the installation and management of Kubernetes applications. It consists of:

- **Helm CLI Tool**: The command-line interface for working with Helm
- **Charts**: Packages of pre-configured Kubernetes resources
- **Repositories**: Collections of charts that can be searched and installed

## Installing Helm

### Downloading Helm
1. Visit the Helm GitHub releases page: [https://github.com/helm/helm/releases](https://github.com/helm/helm/releases)
2. Download the latest stable release for your platform (e.g., `helm-v3.7.1-linux-amd64.tar.gz` for Linux amd64)

### Linux Installation
```bash
# Extract the downloaded archive
tar -xvf helm-v3.7.1-linux-amd64.tar.gz

# Move the binary to a directory in your PATH
sudo mv linux-amd64/helm /usr/local/bin/helm

# Verify installation
helm version
```

## Understanding Helm Charts

### What is a Helm Chart?
A Helm chart is a package that contains all the necessary Kubernetes resource definitions and configurations needed to run an application. It includes:

- `Chart.yaml`: Metadata about the chart
- `values.yaml`: Default configuration values
- `templates/`: Directory of templates that generate Kubernetes manifests
- `charts/`: Directory of dependent charts

### Finding Charts
Charts can be found in various repositories. The primary source is [Artifact Hub](https://artifacthub.io/), which indexes Helm charts from multiple sources.

## Working with Helm Repositories

### Adding Repositories
```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# List configured repositories
helm repo list
```

### Updating Repository Information
```bash
# Update the local Helm chart repository cache
helm repo update
```

### Searching for Charts
```bash
# Search for a specific chart
helm search repo bitnami/mysql

# List all charts in a repository
helm search repo bitnami/
```

## Basic Helm Commands

### Installing a Chart
```bash
# Install a chart with a release name
helm install my-release bitnami/mysql

# Install with auto-generated name
helm install --generate-name bitnami/mysql

# Install with custom values
helm install -f values.yaml my-release bitnami/mysql
```

### Listing Releases
```bash
# List all releases
helm list

# List all releases in all namespaces
helm list --all-namespaces
```

### Uninstalling a Release
```bash
# Uninstall a release
helm uninstall my-release
```

## Practical Example: Installing MySQL

### Basic Installation
```bash
# Add the Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install MySQL
helm install my-mysql bitnami/mysql
```

### Getting Release Information
```bash
# Get release status
helm status my-mysql

# Get release values
helm get values my-mysql

# Get all release information
helm get all my-mysql
```

## Best Practices

1. **Always Check Values**
   - Use `helm show values` to review default values before installation
   - Create a custom `values.yaml` file for your specific needs

2. **Use Namespaces**
   - Install releases in specific namespaces using `-n` or `--namespace`
   - Keeps your cluster organized and resources isolated

3. **Version Control**
   - Store your custom `values.yaml` files in version control
   - Document any customizations made to the default values

4. **Upgrades**
   - Use `helm upgrade` to apply changes to a release
   - Use `--install` to create the release if it doesn't exist

## Common Issues and Troubleshooting

### Permission Issues
```bash
# Check if Tiller is running (Helm 2 only)
kubectl get pods -n kube-system | grep tiller

# Check RBAC permissions
kubectl auth can-i create deployments --as=system:serviceaccount:kube-system:tiller
```

### Connection Issues
```bash
# Check if Helm can connect to the cluster
kubectl cluster-info

# Check Helm configuration
kubectl config view
```

### Rollback a Release
```bash
# View release history
helm history my-release

# Rollback to a previous revision
helm rollback my-release 1
```

## Summary

Helm simplifies the deployment and management of Kubernetes applications by providing a package management system. Key points to remember:

1. Helm uses charts to package applications
2. Repositories store and distribute charts
3. Always review and customize values before installation
4. Use version control for your custom configurations
5. Regularly update your Helm installation and repositories

In the next section, we'll dive deeper into working with Helm charts and customizing them for your needs.

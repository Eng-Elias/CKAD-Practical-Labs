# CKAD Practice Question 4: Helm Operations

## Question
Perform the following Helm operations in the `mercury` namespace:

1. Delete the release `internal-issue-report-v1`
2. Upgrade the release `internal-issue-report-v2` to the latest version of the `bitnami/nginx` chart
3. Install a new release of the `bitnami/apache` chart with the following specifications:
   - Release name: `internal-issue-report-apache`
   - Set replicas to 2
4. Find and delete any broken releases stuck in the "pending-install" state

## Solution

### Prerequisites
Ensure Helm is installed and the Bitnami repository is added:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Task 1: Delete a Release
```bash
# List all releases in the mercury namespace
helm ls -n mercury

# Delete the specified release
helm uninstall internal-issue-report-v1 -n mercury

# Verify deletion
helm ls -n mercury
```

### Task 2: Upgrade a Release
```bash
# Check available chart versions
helm search repo nginx --versions

# Upgrade to the latest version
helm upgrade internal-issue-report-v2 bitnami/nginx -n mercury

# Verify the upgrade
helm ls -n mercury
helm history internal-issue-report-v2 -n mercury
```

### Task 3: Install a New Release with Custom Values
```bash
# Install Apache chart with 2 replicas
helm install internal-issue-report-apache bitnami/apache \
  --namespace mercury \
  --set replicaCount=2

# Verify the installation
helm ls -n mercury
kubectl get deployments -n mercury
```

### Task 4: Clean Up Pending Installs
```bash
# List all releases including pending ones
helm ls -a -n mercury

# Delete any releases stuck in pending-install state
helm uninstall <pending-release-name> -n mercury
```

## Explanation

### Key Concepts
1. **Helm**: A package manager for Kubernetes that allows you to define, install, and upgrade applications.
2. **Releases**: Instances of a chart running in a Kubernetes cluster.
3. **Charts**: Packages of pre-configured Kubernetes resources.
4. **Namespaces**: Virtual clusters within a physical cluster that help organize resources.

### Why These Solutions Work
- `helm uninstall` removes all resources associated with a release
- `helm upgrade` updates a release to a new version of a chart
- `--set` allows overriding default chart values during installation
- `-a` flag with `helm ls` shows all releases including those in a failed state

### Exam Tips
1. **Always verify operations**: After each command, verify the result using `helm ls` or `kubectl` commands.
2. **Use `--dry-run`**: Test installations/upgrades with `--dry-run` first to catch errors.
3. **Check chart values**: Use `helm show values bitnami/chart-name` to see all configurable options.
4. **Namespace awareness**: Always specify the namespace with `-n` to avoid working in the wrong context.
5. **Clean up**: Remove failed releases to prevent conflicts with future operations.

### Common Mistakes to Avoid
- Forgetting to update the Helm repo before installing/upgrading charts
- Not specifying the namespace when working with releases
- Missing the `-a` flag when looking for failed releases
- Not checking resource availability before operations (e.g., enough nodes for replicas)
- Overlooking the need to verify operations after execution

## Additional Practice
1. Rollback a Helm release to a previous version
2. Create a custom values.yaml file for a chart
3. Install a chart with a specific version
4. Debug a failed Helm installation
5. Use `helm template` to see the generated Kubernetes manifests

## Related Commands
```bash
# Add a Helm repository
helm repo add <repo-name> <repo-url>

# List installed charts
helm list -n <namespace>

# Get release history
helm history <release-name> -n <namespace>

# Rollback to a previous version
helm rollback <release-name> <revision> -n <namespace>

# View the values that would be used for installation
helm show values bitnami/apache
```

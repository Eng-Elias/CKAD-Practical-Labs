# 13.2 Working with Helm Charts

## Introduction to Helm Charts

Helm charts are packages of pre-configured Kubernetes resources that can be deployed as a single unit. They simplify the deployment of complex applications by bundling all necessary components and their configurations.

## Basic Chart Structure

A typical Helm chart has the following structure:

```
mychart/
  Chart.yaml          # Metadata about the chart
  values.yaml         # Default configuration values
  charts/             # Directory containing dependent charts
  templates/          # Template files that generate Kubernetes manifests
  templates/tests/    # Test files
  .helmignore        # Files to ignore when packaging the chart
```

## Installing a Chart

### Basic Installation
```bash
# Install with default values
helm install my-release bitnami/mysql

# Install with auto-generated name
helm install --generate-name bitnami/mysql
```

### Verifying the Installation
```bash
# List all releases
helm list

# Check release status
helm status my-release

# View deployed Kubernetes resources
kubectl get all -l app.kubernetes.io/instance=my-release
```

## Customizing Chart Values

### Viewing Default Values
```bash
# View default values for a chart
helm show values bitnami/mysql > values.yaml
```

### Customizing Values
1. Create a custom `values.yaml` file
2. Override the default values as needed
3. Install or upgrade the release with the custom values

```bash
# Install with custom values
helm install -f custom-values.yaml my-release bitnami/mysql

# Upgrade with new values
helm upgrade -f custom-values.yaml my-release bitnami/mysql
```

## Common Chart Operations

### Upgrading a Release
```bash
# Upgrade a release with new values
helm upgrade -f values.yaml my-release bitnami/mysql

# Rollback to a previous version
helm rollback my-release 1
```

### Uninstalling a Release
```bash
# Uninstall a release
helm uninstall my-release

# Keep a record of the release
helm uninstall --keep-history my-release
```

## Working with Chart Repositories

### Adding Repositories
```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# List repositories
helm repo list

# Update repository information
helm repo update
```

### Searching for Charts
```bash
# Search for a chart
helm search repo mysql

# Search in a specific repository
helm search repo bitnami/
```

## Creating Your Own Chart

### Scaffolding a New Chart
```bash
# Create a new chart
helm create mychart

# Directory structure will be created:
# mychart/
#   ├── charts/
#   ├── Chart.yaml
#   ├── templates/
#   └── values.yaml
```

### Key Files in a New Chart

#### Chart.yaml
```yaml
apiVersion: v2
name: mychart
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.16.0"
```

#### values.yaml
```yaml
# Default values for mychart
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "stable"

service:
  type: ClusterIP
  port: 80

resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
```

## Template Functions and Values

### Using Values in Templates
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
```

### Common Template Functions
```yaml
# Default values
replicas: {{ .Values.replicas | default 3 }}

# Required values
database: {{ required "A database name is required" .Values.databaseName }}

# Conditionals
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
{{- end }}

# Loops
{{- range .Values.envVars }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

## Best Practices

### 1. Chart Versioning
- Follow semantic versioning (SemVer) for chart versions
- Increment the version in `Chart.yaml` for each change

### 2. Values Management
- Document all configurable values in `values.yaml`
- Group related values under a common key
- Use descriptive comments for complex configurations

### 3. Templates
- Keep templates simple and readable
- Use named templates for reusable components
- Add validation for required values

## Debugging Helm Charts

### Template Debugging
```bash
# Dry run and render templates
helm install --dry-run --debug my-release ./mychart

# See the generated manifests
helm template my-release ./mychart

# Check for linting issues
helm lint ./mychart
```

### Testing
```bash
# Run chart tests
helm test my-release

# View test pods
kubectl get pods -n <namespace> -l app.kubernetes.io/instance=my-release,app.kubernetes.io/name=mychart-test
```

## Practical Example: Deploying a Custom Application

### 1. Create a New Chart
```bash
helm create myapp
cd myapp
```

### 2. Modify values.yaml
```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.19.10"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 80
```

### 3. Deploy the Chart
```bash
# Install the chart
helm install my-app .

# Verify the deployment
kubectl get all -l app.kubernetes.io/instance=my-app
```

## Summary

Working with Helm charts allows you to package, configure, and deploy applications on Kubernetes efficiently. Key points to remember:

1. Use `helm create` to scaffold new charts
2. Customize deployments using `values.yaml`
3. Use template functions to make charts flexible
4. Follow best practices for versioning and organization
5. Test and debug charts before deploying to production

In the next section, we'll explore Kustomize, another powerful tool for managing Kubernetes configurations.

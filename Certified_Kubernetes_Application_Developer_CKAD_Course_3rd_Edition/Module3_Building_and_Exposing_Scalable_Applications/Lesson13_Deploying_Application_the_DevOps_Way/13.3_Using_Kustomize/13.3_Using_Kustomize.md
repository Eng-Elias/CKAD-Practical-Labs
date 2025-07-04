# 13.3 Using Kustomize

## Introduction to Kustomize

Kustomize is a Kubernetes-native configuration management tool that allows you to customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is. It's built into `kubectl` (since v1.14) and provides a template-free way to customize application configuration.

## Key Concepts

### 1. Base and Overlays
- **Base**: The original set of Kubernetes manifests
- **Overlays**: Customizations applied on top of the base configuration

### 2. Kustomization File
- A file named `kustomization.yaml` that defines how to generate or transform Kubernetes resources
- Can include resources, patches, common labels, namespaces, etc.

## Getting Started

### Basic Directory Structure
```
myapp/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch.yaml
```

### Basic kustomization.yaml
```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: myapp
  environment: base
```

## Basic Commands

### Build Configuration
```bash
# View the generated YAML
kubectl kustomize ./base/

# Apply the configuration
kubectl apply -k ./base/

# Delete the configuration
kubectl delete -k ./base/
```

## Working with Environments

### Development Overlay
```yaml
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

nameSuffix: -dev
commonLabels:
  environment: dev

patches:
- path: patch.yaml
```

### Production Overlay
```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

nameSuffix: -prod
commonLabels:
  environment: prod
  tier: production

replicas:
- name: myapp
  count: 3

images:
- name: myapp
  newName: myregistry.com/myapp
  newTag: v1.0.0
```

## Common Operations

### 1. Adding a Namespace
```yaml
# kustomization.yaml
namespace: my-namespace
resources:
- deployment.yaml
- service.yaml
```

### 2. Adding Common Labels
```yaml
# kustomization.yaml
commonLabels:
  app: myapp
  environment: production
  
resources:
- deployment.yaml
- service.yaml
```

### 3. Adding Common Annotations
```yaml
# kustomization.yaml
commonAnnotations:
  owner: devops-team
  git-commit: abc123
  
resources:
- deployment.yaml
```

## Advanced Features

### 1. ConfigMap and Secret Generation
```yaml
# kustomization.yaml
configMapGenerator:
- name: app-config
  literals:
  - DB_HOST=localhost
  - DB_PORT=5432
  files:
  - config.properties

secretGenerator:
- name: app-secrets
  literals:
  - DB_PASSWORD=secret
  files:
  - secret.properties
  type: Opaque
```

### 2. JSON 6902 Patches
```yaml
# patch.yaml
- op: replace
  path: /spec/replicas
  value: 3
- op: add
  path: /spec/template/spec/containers/0/env
  value:
  - name: ENV
    value: production
```

### 3. Strategic Merge Patches
```yaml
# patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 200m
            memory: 256Mi
```

## Practical Example

### 1. Create Base Configuration
```bash
# Create base directory
mkdir -p myapp/base
cd myapp/base

# Create deployment
cat > deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19.10
        ports:
        - containerPort: 80
EOF

# Create service
cat > service.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp
EOF

# Create kustomization
cat > kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: myapp
  environment: base
EOF
```

### 2. Create Development Overlay
```bash
# Create overlay directory
mkdir -p ../overlays/dev
cd ../overlays/dev

# Create patch
cat > patch.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: myapp
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 64Mi
        env:
        - name: ENV
          value: development
EOF

# Create kustomization
cat > kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

nameSuffix: -dev
namespace: dev

commonLabels:
  environment: dev

patches:
- path: patch.yaml

images:
- name: nginx
  newTag: 1.19.10

replicas:
- name: myapp
  count: 1
EOF
```

### 3. Apply Configuration
```bash
# Build and apply development configuration
kubectl apply -k overlays/dev/

# Verify resources
kubectl get all -n dev
```

## Kustomize vs. Helm

| Feature               | Kustomize                          | Helm                               |
|-----------------------|-----------------------------------|-----------------------------------|
| **Approach**         | Declarative, template-free        | Templating-based                 |
| **Learning Curve**   | Lower                             | Higher                           |
| **Complexity**       | Simpler                           | More complex                     |
| **Native Support**   | Built into kubectl                | Requires separate installation   |
| **Best For**         | Simple customizations             | Complex package management       |
| **Versioning**       | Git-based                         | Chart repositories               |

## Best Practices

1. **Directory Structure**
   - Keep base configurations minimal and reusable
   - Use overlays for environment-specific customizations
   - Follow a consistent naming convention

2. **Kustomization Files**
   - Keep `kustomization.yaml` files small and focused
   - Use patches for complex modifications
   - Document non-obvious customizations

3. **Version Control**
   - Store base and overlays in version control
   - Use tags or branches for different environments
   - Document the relationship between bases and overlays

## Common Issues and Solutions

### 1. Resource Not Found
```bash
# Error: accumulating resources: accumulateFile "accumulating resources from 'deployment.yaml':
# evalsymlink failure on '/path/to/deployment.yaml' : lstat /path/to/deployment.yaml: no such file or directory"

# Solution: Ensure all resource paths in kustomization.yaml are correct
```

### 2. Invalid Patch
```bash
# Error: json: cannot unmarshal string into Go value of type types.Patch

# Solution: Ensure your patch file uses the correct format (YAML or JSON 6902)
```

### 3. Field Not Found
```bash
# Error: couldn't find key app in Secret

# Solution: Verify that the resource exists and has the specified field
```

## Summary

Kustomize provides a powerful way to manage Kubernetes configurations without templates. Key points to remember:

1. Use bases for common configurations
2. Create overlays for environment-specific customizations
3. Leverage patches for targeted modifications
4. Use generators for ConfigMaps and Secrets
5. Follow best practices for maintainability

Kustomize is particularly useful when you need to manage multiple similar environments with slight variations, making it a valuable tool in your Kubernetes configuration management toolkit.

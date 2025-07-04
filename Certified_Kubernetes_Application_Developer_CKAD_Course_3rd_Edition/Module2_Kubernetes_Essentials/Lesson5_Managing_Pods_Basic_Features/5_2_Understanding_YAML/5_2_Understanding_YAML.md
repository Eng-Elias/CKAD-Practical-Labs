# 5.2 Understanding YAML

## What is YAML?
YAML (YAML Ain't Markup Language) is a human-readable data serialization language that is commonly used for configuration files. In Kubernetes, YAML is the primary format for defining and managing resources.

### Key Features of YAML
- **Human-readable**: Easy to read and write
- **Structured format**: Uses indentation to represent hierarchy
- **Case sensitive**: Keys and values are case-sensitive
- **Comments**: Supports single-line comments starting with `#`
- **Multiple documents**: Multiple YAML documents can be in a single file separated by `---`

## Basic YAML Syntax

### Key-Value Pairs
```yaml
key: value
```

### Lists/Arrays
```yaml
fruits:
  - apple
  - banana
  - orange
```

### Dictionaries/Maps
```yaml
person:
  name: John Doe
  age: 30
  address:
    street: 123 Main St
    city: Anytown
```

## Kubernetes YAML Structure

### Required Fields
Every Kubernetes YAML file contains these top-level fields:

```yaml
apiVersion: v1        # API version to use
kind: Pod             # Type of resource
metadata:             # Identifying information
  name: my-pod        # Name of the resource
spec:                 # Desired state of the resource
  # Resource-specific configuration
```

### Common Kubernetes YAML Components

#### Metadata
```yaml
metadata:
  name: my-pod
  labels:
    app: nginx
    tier: frontend
  annotations:
    description: "This is a test pod"
```

#### Container Specification
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## Working with kubectl explain

### Exploring API Resources
```bash
# List all API resources
kubectl api-resources

# Explain a specific resource
kubectl explain pod

# Get detailed information about a specific field
kubectl explain pod.spec

# Get recursive explanation of all fields
kubectl explain pod --recursive
```

## Creating Resources from YAML

### Creating a Resource
```bash
kubectl create -f pod.yaml
```

### Applying Changes
```bash
kubectl apply -f pod.yaml
```

### Deleting a Resource
```bash
kubectl delete -f pod.yaml
```

## YAML Best Practices

### 1. Use Consistent Indentation
- Use 2 spaces for indentation (not tabs)
- Be consistent with spacing and alignment

### 2. Use Meaningful Names
- Choose descriptive names for resources
- Use lowercase with hyphens for resource names

### 3. Keep YAML Files Modular
- Split complex configurations into multiple files
- Use Kustomize or Helm for managing multiple YAML files

### 4. Use Comments
- Add comments to explain complex configurations
- Document the purpose of each section

### 5. Validate YAML
- Use `kubectl apply --dry-run=client -f pod.yaml` to validate
- Use YAML linters in your IDE

## Common YAML Issues

### 1. Indentation Errors
```yaml
# Wrong
spec:
containers:  # Missing indentation
- name: nginx

# Correct
spec:
  containers:
  - name: nginx
```

### 2. Type Confusion
```yaml
# Wrong (number as string)
port: "80"

# Correct
port: 80
```

### 3. Boolean Values
```yaml
# Wrong
privileged: "true"

# Correct
privileged: true
```

## Advanced YAML Features

### Multi-document YAML
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
---
apiVersion: v1
kind: Service
metadata:
  name: service1
```

### Anchors and Aliases
```yaml
defaults: &defaults
  image: nginx:1.14.2
  restartPolicy: Always

pod1:
  <<: *defaults
  name: pod1

pod2:
  <<: *defaults
  name: pod2
```

## Next Steps
- Practice creating different Kubernetes resources using YAML
- Learn about generating YAML files using `kubectl`
- Explore more complex pod configurations
- Understand how to use Kustomize for managing multiple YAML files

# 3.4 Understanding Kubernetes Management Interfaces

## Kubernetes API Overview

- **API (Application Programming Interface)**: Standardized interface between client and server
- **Purpose**: Provides programmatic access to all Kubernetes functionality
- **Key Characteristics**:
  - RESTful design
  - Extensible
  - Versioned
  - Self-documenting

## Primary Management Interfaces

### 1. kubectl (Command Line Interface)

#### Key Features
- Primary interface for Kubernetes administration
- Provides complete control over Kubernetes clusters
- Essential for CKAD certification

#### Basic Commands
```bash
# Get help
kubectl --help
kubectl -h

# View resources
kubectl get all
kubectl get pods
kubectl get services

# Create resources
kubectl create deployment nginx --image=nginx

# Describe resources
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
```

### 2. Direct API Access

#### When to Use
- Advanced scenarios
- Custom automation
- Debugging and troubleshooting

#### Example with curl
```bash
# Get API versions
curl -k https://<cluster-ip>/api/

# Get pods
curl -k https://<cluster-ip>/api/v1/namespaces/default/pods
```

### 3. Web Dashboard (Not CKAD Focus)

#### Key Points
- Not covered in CKAD exam
- Implementation varies by distribution
- Useful for visualization but not for exam preparation

## kubectl in Depth

### Context and Configuration
```bash
# View current context
kubectl config current-context

# List available contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>
```

### Common kubectl Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-n`, `--namespace` | Specify namespace | `kubectl get pods -n kube-system` |
| `-o`, `--output` | Output format | `kubectl get pods -o wide` |
| `-l`, `--selector` | Label selector | `kubectl get pods -l app=nginx` |
| `--all-namespaces` | All namespaces | `kubectl get pods --all-namespaces` |

### Output Formats
```bash
# JSON output
kubectl get pods -o json

# YAML output
kubectl get pod <pod-name> -o yaml

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

## Best Practices

1. **Use Configuration Files**
   - Store configurations in YAML files
   - Version control your configurations
   - Use `kubectl apply -f <file>.yaml`

2. **Use Namespaces**
   - Organize resources logically
   - Isolate environments (dev, staging, prod)
   - Apply resource quotas

3. **Use Labels and Selectors**
   - Tag resources meaningfully
   - Enable efficient resource management
   - Simplify operations and automation

## Common kubectl Shortcuts

| Command | Description |
|---------|-------------|
| `kubectl get po` | Get pods |
| `kubectl get svc` | Get services |
| `kubectl get deploy` | Get deployments |
| `kubectl get ns` | Get namespaces |
| `kubectl describe` | Show detailed information |
| `kubectl logs -f` | Stream logs |

## Next Steps
- Explore `kubectl` plugins
- Learn about `kustomize` for template management
- Practice with `kubectl` dry-run and diff

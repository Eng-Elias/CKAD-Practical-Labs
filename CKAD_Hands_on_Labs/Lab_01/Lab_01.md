# CKAD Lab 1: Creating Multiple Resources from a Directory

## Objective
Learn how to create multiple Kubernetes resources simultaneously from a directory containing all the required YAML manifest files.

## The Scenario
Imagine you have a directory filled with dozens or even hundreds of YAML files, each defining a Kubernetes resource like a Pod, Service, ConfigMap, or Secret. Applying them one by one would be inefficient.

**Example Directory Structure:**
Let's assume you have a directory named `demo` with the following files:
```
demo/
├── config-map.yaml
├── network-policy.yaml
├── pod.yaml
├── secret.yaml
└── service-account.yaml
```

## Solution
The solution is to use the `kubectl apply -f` command and point it to the directory instead of a single file. `kubectl` will automatically read and apply all valid manifest files within that directory.

### Command
```bash
# Apply all YAML files in the 'demo' directory
kubectl apply -f demo
```

### Expected Output
Kubernetes will confirm the creation of each resource found in the directory's files.
```
serviceaccount/my-sa created
configmap/my-config created
secret/my-secret created
networkpolicy.networking.k8s.io/my-netpol created
pod/my-pod created
```

### Verification
You can then verify that the individual resources have been created in the cluster.
```bash
# Check for the pod
kubectl get pod

# Check for the secret
kubectl get secret

# Check for the service account
kubectl get sa
```

## Explanation
The `kubectl apply -f <directory>` command is a fundamental and highly efficient feature for managing Kubernetes applications. It simplifies the deployment of complex applications that are composed of many different resource types by treating them as a single group. This is a crucial command for both development and CI/CD pipelines.

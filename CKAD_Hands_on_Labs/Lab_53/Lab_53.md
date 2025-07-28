# CKAD Lab 53: Creating Multiple Resources from a Single YAML File

## Objective
Learn how to define multiple Kubernetes resources in a single YAML file. This is a common and convenient practice for grouping related resources, ensuring they can be created and managed as a single unit.

## The Challenge
Create a `Pod` and a `ConfigMap` at the same time by applying a single YAML file.

## The Solution: The YAML Document Separator
The key to defining multiple resources in one file is the YAML document separator, which is a line containing only three dashes (`---`).

When `kubectl apply` processes a file, it treats each section separated by `---` as a distinct resource manifest.

### Step 1: The Combined Manifest
To create a `Pod` and a `ConfigMap` together, you simply place their respective YAML definitions in the same file, separated by `---`.

**`multi-resource.yaml`:**
```yaml
# Definition for the first resource: a ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  app.properties: |
    greeting=hello
    user=world
---
# The separator marks the beginning of a new resource definition
# Definition for the second resource: a Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

### Step 2: What Happens Without the Separator?
If you forget the `---`, `kubectl` will try to parse the entire file as a single resource. This will fail with a confusing error message, often complaining about an unexpected field from the second resource's definition.

For example: `error: error validating "multi-resource.yaml": error validating data: found unknown field "spec" in v1.ConfigMap;`

This error occurs because after parsing the `ConfigMap`, `kubectl` encounters the `spec` field from the `Pod` definition and doesn't know what to do with it in the context of a `ConfigMap`.

### Step 3: Apply and Verify
With the separator in place, the apply command will create both resources successfully.

**Commands:**
```bash
kubectl apply -f multi-resource.yaml
# configmap/my-config created
# pod/my-pod created

# Verify that both resources exist
kubectl get configmap my-config
kubectl get pod my-pod
```

## Exam Tip
While this might seem like a simple concept, it's a powerful technique for organizing your work during the CKAD exam. If a question requires you to create a `Deployment` and expose it with a `Service`, you can write both manifests in a single file separated by `---`. This keeps your work organized and ensures you don't forget to create one of the required components.

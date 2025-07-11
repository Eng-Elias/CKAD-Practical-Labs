# CKAD Lab 6: Understanding Kubernetes Annotations

## Objective
Learn what Kubernetes annotations are, how they differ from labels, and how to attach them to resources like pods.

## What are Annotations?
Annotations are key-value pairs used to attach arbitrary, non-identifying metadata to Kubernetes objects. Unlike labels, annotations are not used by Kubernetes to select or organize objects. Instead, they are primarily for consumption by humans or external tools and libraries.

Common use cases for annotations include:
- Storing build, release, or image information (e.g., build number, git commit hash).
- Contact information for the person responsible for the resource.
- Pointers to logging, monitoring, or analytics dashboards.
- Instructions for client-side libraries or tooling (e.g., configuration for an Ingress controller or a service mesh like Istio).

## Solution
This lab demonstrates how to create a pod with custom annotations defined in its YAML manifest.

### Step 1: Create a YAML File with Annotations
Create a YAML file, for example `pod-with-annotations.yaml`, and define the pod. Inside the `metadata` section, add an `annotations` block with your desired key-value pairs.

**`pod-with-annotations.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-foo
  annotations:
    # Custom annotations for external tools or information
    description: "This is a demo pod for annotations"
    contact: "admin@example.com"
    build-version: "1.3.4"
spec:
  containers:
  - name: nginx
    image: nginx
```

### Step 2: Apply the Manifest
Create the pod using `kubectl apply`.

**Command:**
```bash
kubectl apply -f pod-with-annotations.yaml
```

### Step 3: Verify the Annotations
You can view the annotations of a running resource using `kubectl describe` or by getting the full YAML/JSON output.

**Command:**
```bash
# Use 'describe' to see the annotations in a human-readable format
kubectl describe pod pod-foo
```

**Expected Output from `describe`:**
```
Name:         pod-foo
Namespace:    default
...
Annotations:  build-version: 1.3.4
              contact: admin@example.com
              description: This is a demo pod for annotations
...
```

## Explanation

As you can see, the annotations are stored as part of the pod's metadata but do not influence the pod's lifecycle or scheduling. They are simply extra pieces of information attached to the object.

It's also common to see annotations added by Kubernetes itself or other tools. For example, when you use `kubectl apply`, Kubernetes often adds a `kubectl.kubernetes.io/last-applied-configuration` annotation to track the object's state, which is essential for declarative management.

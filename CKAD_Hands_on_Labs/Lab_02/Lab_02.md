# CKAD Lab 2: Getting Resource Configurations in Different Formats

## Objective
Learn how to retrieve the full configuration of live Kubernetes resources in various formats, such as YAML and JSON. This is a fundamental skill for inspecting running resources and for creating new manifests from existing ones.

## Key Commands

The `-o` (or `--output`) flag in `kubectl get` allows you to specify the output format.

### 1. Get Resource as YAML
To get the complete YAML definition of a running resource, use `-o yaml`. This is extremely useful for backing up a resource's state or for creating a new, similar resource.

**Command:**
```bash
# Get the YAML for a running pod named 'my-pod'
kubectl get pod my-pod -o yaml

# Get the YAML for a service named 'my-service'
kubectl get service my-service -o yaml
```

**Example Output (for a pod):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-10-27T12:00:00Z"
  name: my-pod
  namespace: default
  ...
spec:
  containers:
  - image: nginx
    name: nginx
    ...
status:
  ...
```

### 2. Get Resource as JSON
Similarly, you can retrieve the configuration in JSON format using `-o json`. This is useful for programmatic parsing or integration with tools that consume JSON.

**Command:**
```bash
# Get the JSON for a running pod named 'my-pod'
kubectl get pod my-pod -o json
```

**Tip**: Pipe the output to `jq` for a more readable, "prettified" format.
```bash
kubectl get pod my-pod -o json | jq
```

### 3. Get More Details with `-o wide`
When listing multiple resources, the default output is a summary. Using `-o wide` provides additional information, such as the pod's IP address and the node it's running on.

**Command:**
```bash
# List all pods with additional details
kubectl get pods -o wide

# List all nodes with additional details
kubectl get nodes -o wide
```

**Example Output (for pods):**
```
NAME     READY   STATUS    RESTARTS   AGE   IP          NODE           NOMINATED NODE   READINESS GATES
my-pod   1/1     Running   0          15m   10.244.1.5   gke-worker-1   <none>           <none>
```

## Explanation

Being able to extract the configuration of a live resource is a critical skill in Kubernetes management. 

-   **`-o yaml`** is your go-to for creating a template for a new resource. You can redirect the output to a file (`> my-pod.yaml`), edit it (e.g., change the name and a few properties), and then use `kubectl apply -f my-pod.yaml` to create a new resource.
-   **`-o json`** is primarily for automation and scripting.
-   **`-o wide`** is essential for quickly diagnosing issues, like checking which node a pod is scheduled on or finding its IP for direct communication tests.

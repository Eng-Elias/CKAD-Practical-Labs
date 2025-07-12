# CKAD Lab 12: Getting Logs from Specific Pods and Containers

## Objective
Reinforce the use of `kubectl logs` and learn how to handle two common scenarios: getting logs from a pod in a specific namespace and getting logs from a specific container within a multi-container pod.

## The `kubectl logs` Command
As covered previously, `kubectl logs` is the primary tool for inspecting the output of your applications running in pods. This lab focuses on the flags required for more specific targeting.

## Solution

### 1. Getting Logs from a Pod in a Different Namespace
If the pod you want to inspect is not in your current context's default namespace, you must specify its namespace using the `-n` (or `--namespace`) flag.

**Command:**
```bash
# Get logs from 'my-pod' which is running in the 'production' namespace
kubectl logs my-pod -n production
```
Failure to provide the namespace will result in an error stating that the pod was not found in the default namespace.

### 2. Getting Logs from a Specific Container in a Pod
If a pod contains more than one container (e.g., an application container and a logging sidecar), `kubectl` does not know which container's logs to show by default. You must specify the container name using the `-c` (or `--container`) flag.

**Command:**
```bash
# A pod named 'my-multi-container-pod' has two containers: 'app' and 'sidecar'

# Get logs from the 'app' container
kubectl logs my-multi-container-pod -c app

# Get logs from the 'sidecar' container
kubectl logs my-multi-container-pod -c sidecar
```

**Error Scenario:**
If you try to get logs from a multi-container pod without the `-c` flag, `kubectl` will return an error listing the available containers.
```bash
# Command
kubectl logs my-multi-container-pod

# Expected Error
Error from server (BadRequest): a container name must be specified for pod my-multi-container-pod, choose one of: [app, sidecar]
```

## Explanation

These two flags, `-n` and `-c`, are essential for using `kubectl logs` effectively in any real-world Kubernetes environment.

-   **`-n <namespace>`**: Always be mindful of the namespace you are working in. Forgetting to specify the namespace is one of the most common sources of "resource not found" errors.
-   **`-c <container>`**: As the use of sidecar patterns (for service meshes, logging, etc.) becomes more common, you will frequently encounter pods with multiple containers. Knowing how to target a specific one for debugging is a necessary skill.

# CKAD Lab 54: Finding a Container's Environment Variables

## Objective
Learn two different methods to inspect the environment variables of a container running inside a pod. This is a fundamental skill for debugging applications and verifying configurations in Kubernetes.

## The Challenge
A pod named `foo` is running somewhere in the cluster. 
1.  Find which namespace the pod belongs to.
2.  Find the names and values of its environment variables.

## Solution

### Step 1: Find the Pod and its Namespace
When you don't know the namespace, you can search across all of them using the `-A` or `--all-namespaces` flag.

**Command:**
```bash
kubectl get pods -A
```

Look for the pod named `foo` in the output to identify its namespace.

### Method 1: Using `kubectl describe`
This is the quickest way to see the configuration of a pod as the API server sees it. The `describe` command provides a detailed overview, including the `Environment` section for each container.

**Command:**
```bash
# Replace <namespace> with the namespace you found in Step 1
kubectl describe pod foo -n <namespace>
```

Scroll through the output to the `Containers` section. You will find an `Environment` block that lists all the environment variables and their values.

**Example Output Snippet:**
```
Containers:
  ...
  Environment:
    FOO_PORT:  8080
  Mounts:
    ...
```

### Method 2: Using `kubectl exec`
This method allows you to run a command directly inside the container. By running the `env` command, you can see the environment as the running process sees it.

**Command:**
```bash
# Get an interactive shell inside the container
kubectl exec -it foo -n <namespace> -- /bin/sh

# Once inside the container, run the 'env' command
# (The prompt will change to something like 'root@foo:/$')
env
```

The output of `env` will list all environment variables, including the custom ones set for the application and the standard ones injected by Kubernetes.

## Exam Tip: Multi-Container Pods
This is a critical point. If a pod has **more than one container**, both `kubectl describe` and `kubectl exec` will default to the **first container** defined in the manifest.

If the environment variable you are looking for is in a different container, you **must** specify the container's name using the `-c <container-name>` flag.

**Example:**
```bash
# Exec into a specific container named 'sidecar'
kubectl exec -it foo -n <namespace> -c sidecar -- /bin/sh
```
Forgetting the `-c` flag in a multi-container scenario is a common mistake. Always be aware of how many containers are in the pod you are inspecting.

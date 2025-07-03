# 15.3 Working with Namespaces

This section provides a solution for the first sample exam assignment, which focuses on creating and managing resources within a specific namespace.

## Task

1.  Create a namespace named `ckad-ns1` in your cluster.
2.  In this namespace, run the following pods:
    *   A pod named `pod-a` running the `httpd` server image.
    *   A pod named `pod-b` running both the `nginx` server image and the `alpine` image.

---

## Solution Walkthrough

### 1. Create the Namespace

First, create the required namespace.

```bash
kubectl create ns ckad-ns1
```

Verify its creation:

```bash
kubectl get ns
```

### 2. Create Pod A (Single Container)

Next, create the first pod. The key here is to specify the correct namespace.

```bash
kubectl run pod-a --image=httpd -n ckad-ns1
```

**Exam Tip:** Always explicitly specify the namespace with the `-n` flag for every resource. If you create a resource in the wrong namespace (e.g., `default`), you will get zero points for the task. Avoid changing your default context as it's easy to forget and make mistakes on subsequent tasks.

### 3. Create Pod B (Multi-Container)

Creating a multi-container pod directly from the command line is complex. A better approach is to generate a base YAML file and then modify it.

#### Generate Base YAML

Generate the YAML for a pod with one of the required containers (`alpine`). We use `--dry-run=client -o yaml` to output the resource definition without creating it.

```bash
# Note: The sleep command is a placeholder to keep the container running.
# The command and its arguments must come after a `--` separator at the end of the command.
kubectl run pod-b --image=alpine -n ckad-ns1 --dry-run=client -o yaml -- sleep 3600 > task1.yaml
```

**Exam Tip:** On the exam, it's a good practice to name your YAML files after the task number (e.g., `task1.yaml`) for easy reference and verification.

#### Modify the YAML

Now, edit the generated `task1.yaml` to add the second container (`nginx`).

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-b
  name: pod-b
  namespace: ckad-ns1
spec:
  containers:
  # First container, generated from the command
  - image: alpine
    name: alpine # Changed from pod-b for clarity
    args:
    - sleep
    - "3600"
    resources: {}
  # Manually add the second container
  - name: nginx
    image: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### Create the Pod from YAML

Apply the configuration from the YAML file.

```bash
kubectl create -f task1.yaml
```

If you encounter an error like `pod "pod-b" already exists` (perhaps from a previous attempt), simply delete the existing pod and re-apply the YAML.

```bash
kubectl delete pod pod-b -n ckad-ns1
kubectl create -f task1.yaml
```

### 4. Verify the Solution

Finally, verify that both pods are running correctly in the `ckad-ns1` namespace.

Check the status of the pods. `pod-b` should show `2/2` containers ready.

```bash
kubectl get pods -n ckad-ns1
```

Describe `pod-b` to confirm that both the `alpine` and `nginx` containers are present and running.

```bash
kubectl describe pod pod-b -n ckad-ns1
```

This confirms the correct solution for the assignment.

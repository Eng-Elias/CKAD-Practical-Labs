# CKAD Lab 47 (Part 1): Creating a Multi-Container Pod with Environment Variables

## Objective
This two-part lab covers how to create a single pod that runs multiple containers, with each container having its own distinct environment variables. This is a common pattern for "sidecar" containers or for grouping closely related processes together.

## The Challenge
Create a single pod named `foo-pod` with the following specifications:

-   **Container 1:**
    -   Name: `nginx-container`
    -   Image: `nginx`
    -   Environment Variable: `MY_VAR=foo`
-   **Container 2:**
    -   Name: `busybox-container`
    -   Image: `busybox`
    -   Command: `sleep 3600`
    -   Environment Variable: `MY_VAR=bar`

## Part 1 Solution: Generating the Template
Since a pod manifest with multiple containers can be verbose, the fastest approach is to generate a template for a single-container pod and then edit the resulting YAML file.

### Step 1: Generate a Base YAML Manifest
We can use `kubectl run` with the `--dry-run=client -o yaml` flags to generate the YAML for a simple pod without actually creating it in the cluster.

**Command:**
```bash
kubectl run foo-pod --image=nginx --dry-run=client -o yaml > pod-multi.yaml
```

**Command Breakdown:**
-   `kubectl run foo-pod --image=nginx`: The basic command to create a pod.
-   `--dry-run=client`: Tells `kubectl` to generate the object on the client-side without sending it to the API server.
-   `-o yaml`: Specifies that the output format should be YAML.
-   `> pod-multi.yaml`: Redirects the output into a file named `pod-multi.yaml`.

### Step 2: The Initial Template
The `pod-multi.yaml` file will look something like this:

**`pod-multi.yaml` (Initial Template):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: foo-pod
  name: foo-pod
spec:
  containers:
  - image: nginx
    name: foo-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
This gives us a valid starting point with one container already defined.

## Next Steps
In Part 2, we will edit this manifest to:
1.  Add the environment variable to the first container.
2.  Add the complete definition for the second `busybox` container.
3.  Apply the final manifest to create the multi-container pod.

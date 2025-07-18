# CKAD Lab 26 (Part 1): Practice Exam - Namespace, ConfigMap, and Pod Setup

## Objective
Begin a multi-part practice lab that simulates a real CKAD exam question. This part covers creating a namespace and a ConfigMap imperatively, and then preparing a pod manifest to add an `emptyDir` volume.

## The Full Challenge (Multi-Part)

1.  Create a namespace named `foo-ns`.
2.  Create a ConfigMap named `foo-cm` in the `foo-ns` namespace with the data `foo=bar`.
3.  Create a Pod named `foo-pod` in the `foo-ns` namespace using the `busybox` image.
4.  The pod must have a volume of type `emptyDir` mounted at `/etc/foo`.
5.  The pod should run a command that continuously writes the value of the `foo` key from the `foo-cm` ConfigMap into the file `/etc/foo/foo.txt`.

This lab tests your knowledge of namespaces, imperative commands, ConfigMaps, volumes, and shell commands within a pod.

## Part 1 Solution

### Step 1: Create the Namespace
Use a simple imperative command to create the required namespace.

**Command:**
```bash
kubectl create namespace foo-ns
```

### Step 2: Create the ConfigMap
Create the ConfigMap using the `--from-literal` flag. It's crucial to specify the correct namespace using the `-n` flag so the ConfigMap is created in the right place.

**Command:**
```bash
kubectl create configmap foo-cm --from-literal=foo=bar -n foo-ns
```

### Step 3: Generate the Base Pod Manifest
Use `kubectl run` with `--dry-run=client -o yaml` to generate the starting YAML for the pod. This is much faster than writing it from scratch.

**Command:**
```bash
kubectl run foo-pod --image=busybox -n foo-ns --dry-run=client -o yaml > pod.yaml
```

This command creates a `pod.yaml` file. Note that because we used `-n foo-ns` in the command, the namespace will already be correctly set in the manifest's metadata.

### Step 4: Add the `emptyDir` Volume and Mount
This is the step where the video cuts off. To add a volume, you need to define it in two places in the pod spec:
1.  **`spec.volumes`:** This defines the volume at the pod level.
2.  **`spec.containers[0].volumeMounts`:** This tells the container where to mount the defined volume.

Here is how you would edit the `pod.yaml` file to add the `emptyDir` volume:

**`pod.yaml` (prepared for Part 2):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
  namespace: foo-ns
spec:
  containers:
  - image: busybox
    name: foo-pod
    resources: {}
    # This section mounts the volume inside the container
    volumeMounts:
    - name: my-volume # This name must match the volume name below
      mountPath: /etc/foo
  
  # This section defines the volume for the pod
  volumes:
  - name: my-volume
    emptyDir: {} # An empty object means a standard emptyDir volume

  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

This file is now ready for Part 2, where we will add the command to use the ConfigMap and write to the mounted volume.

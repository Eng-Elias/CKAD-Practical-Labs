# CKAD Lab 15: Using an Init Container with a Shared Volume

## Objective
Learn what an Init Container is and how to use one with a shared `emptyDir` volume to perform prerequisite tasks before the main application container starts.

## What are Init Containers?
Init Containers are special containers that run to completion before the main application containers in a Pod are started. If any of the Init Containers fail, Kubernetes restarts the Pod until the Init Container succeeds.

They are perfect for performing setup tasks that must complete before the main application is ready, such as:
-   Waiting for a database or another service to be available.
-   Downloading configuration files or assets from a remote source.
-   Performing database schema migrations.
-   Setting up necessary permissions on a shared volume.

## The `emptyDir` Volume
An `emptyDir` volume is a simple, temporary storage volume that is created when a Pod is assigned to a node. It is initially empty and exists as long as that Pod is running on that node. All containers within the Pod can read and write the same files in the `emptyDir` volume, making it the perfect way for an Init Container to share data with an application container.

## Solution
This lab demonstrates a common pattern: an Init Container downloads a web page (`index.html`) into a shared `emptyDir` volume, and the main `nginx` container then serves that page.

### Step 1: Create the Pod Manifest
Create a YAML file named `pod-with-init.yaml`. This manifest defines:
1.  An `emptyDir` volume named `workdir`.
2.  An `initContainers` block with a `busybox` container. This container mounts the `workdir` volume and runs a `wget` command to download `index.html` into it.
3.  A main `containers` block with an `nginx` container. This container also mounts the `workdir` volume, but at the location where `nginx` serves its web content (`/usr/share/nginx/html`).

**`pod-with-init.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  # Define the Init Container
  initContainers:
  - name: install
    image: busybox
    # This command downloads the index.html file into the shared volume
    command: ['wget', '-O', '/work-dir/index.html', 'http://info.cern.ch']
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"

  # Define the main application container
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      # Mount the volume at the nginx web root
      mountPath: /usr/share/nginx/html

  # Define the shared volume
  volumes:
  - name: workdir
    emptyDir: {}
```

### Step 2: Apply the Manifest and Verify
Create the pod and then `exec` into the running `nginx` container to verify that the `index.html` file created by the Init Container is present.

**Commands:**
```bash
# Create the pod
kubectl apply -f pod-with-init.yaml

# Wait for the pod to be in the 'Running' state
kubectl get pods --watch

# Exec into the nginx container and check for the file
kubectl exec -it my-app-pod -- ls /usr/share/nginx/html
# Expected output: index.html

# View the contents of the file
kubectl exec -it my-app-pod -- cat /usr/share/nginx/html/index.html
```

## Explanation
This pattern is extremely powerful because it separates the one-time setup logic from the main application logic. The `nginx` container image doesn't need to know *how* the `index.html` file got there; it only needs to serve it. The Init Container handles the "how." The `emptyDir` volume acts as the bridge, allowing the two containers to communicate and share files seamlessly within the lifecycle of the Pod.

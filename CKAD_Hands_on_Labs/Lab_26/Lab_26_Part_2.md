# CKAD Lab 26 (Part 2): Practice Exam - Configuring the Pod

## Objective
Continue the practice lab by configuring the pod's container. This involves injecting the ConfigMap data as an environment variable and setting up a command to write that data to the mounted volume.

## Recap of the Challenge

-   **Namespace:** `foo-ns` (Created)
-   **ConfigMap:** `foo-cm` with `foo=bar` (Created)
-   **Pod:** `foo-pod` with `busybox` image.
-   **Volume:** `emptyDir` mounted at `/etc/foo` (Prepared in Part 1).
-   **Task:** The pod must run a command that continuously writes the value from the ConfigMap into the file `/etc/foo/foo.txt`.

## Part 2 Solution

We will now edit the `pod.yaml` file from Part 1 to add the final two pieces of configuration: the environment variable and the container command.

### Step 1: Injecting the ConfigMap Value as an Environment Variable

To get the value of the `foo` key from our `foo-cm` ConfigMap, we need to add an `env` section to our container definition. We will use `valueFrom.configMapKeyRef` to specify the exact source of the data.

**YAML Snippet for the Environment Variable:**
```yaml
    env:
    - name: MY_FOO_VAR # The name of the env var inside the pod
      valueFrom:
        configMapKeyRef:
          name: foo-cm   # The name of the ConfigMap
          key: foo      # The key within the ConfigMap
```

### Step 2: Adding the Container Command

The final requirement is to run a command that writes the value of our new environment variable (`$MY_FOO_VAR`) to the file `/etc/foo/foo.txt` in a continuous loop.

We use the standard `command` and `args` pattern for running a shell command.

**YAML Snippet for the Command:**
```yaml
    command: ["/bin/sh", "-c"]
    args:
    - |
      while true; do
        echo "The value from the configmap is: $MY_FOO_VAR" >> /etc/foo/foo.txt;
        sleep 5;
      done
```
**Note:** Using the `|` character allows for a multi-line, readable shell script in the YAML.

### Step 3: Assembling the Complete Pod Manifest

Now, let's combine the work from Part 1 and Part 2 into the final, complete `pod.yaml`.

**Final `pod.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
  namespace: foo-ns
spec:
  containers:
  - name: foo-container
    image: busybox
    # 1. Define the command to run
    command: ["/bin/sh", "-c"]
    args:
    - |
      while true; do
        echo "The value from the configmap is: $MY_FOO_VAR" >> /etc/foo/foo.txt;
        sleep 5;
      done
    # 2. Inject the ConfigMap key as an environment variable
    env:
    - name: MY_FOO_VAR
      valueFrom:
        configMapKeyRef:
          name: foo-cm
          key: foo
    # 3. Mount the volume
    volumeMounts:
    - name: my-volume
      mountPath: /etc/foo
    resources: {}

  # 4. Define the volume
  volumes:
  - name: my-volume
    emptyDir: {}

  restartPolicy: OnFailure # Use OnFailure or Never for tasks; Always is for services
```

This manifest is now complete and ready to be deployed. Part 3 will cover applying this manifest and verifying that all requirements have been met.

# CKAD Lab 45 (Part 1): Configuring a `livenessProbe`

## Objective
This two-part lab covers how to configure a `livenessProbe`. While a `readinessProbe` determines if a container is ready to receive traffic, a `livenessProbe` determines if a container is still running correctly. If a `livenessProbe` fails, the kubelet will kill the container and restart it according to its `restartPolicy`.

## The Challenge
Create a pod named `foo-pod` with an `nginx` container and the following `livenessProbe` configuration:
-   **Probe Type:** `exec` (execute a command)
-   **Command:** `ping -c 2 8.8.8.8`
-   **Initial Delay:** `20` seconds
-   **Period:** `30` seconds (run the probe every 30 seconds)

## Part 1 Solution: Building the Manifest
The structure for a `livenessProbe` is very similar to a `readinessProbe`. We can often find a good starting point in the official Kubernetes documentation and adapt it.

### Step 1: The `livenessProbe` Structure
The probe is defined within the container's specification. For a command-based probe, we use the `exec` field.

**`pod-liveness.yaml` (Initial Draft):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
spec:
  containers:
  - name: nginx-liveness
    image: nginx
    # The probe checks if the container is still healthy
    livenessProbe:
      # Use the 'exec' field for a command-based probe
      exec:
        # The command and its arguments are specified as a list of strings
        command:
        - "ping"
        - "-c"
        - "2"
        - "8.8.8.8"
      # Wait 20s before the first probe
      initialDelaySeconds: 20
      # Run the probe every 30s
      periodSeconds: 30
```

**Explanation of Key Fields:**
-   `livenessProbe`: The top-level object for the probe configuration.
-   `exec.command`: An array of strings representing the command to be executed inside the container. A non-zero exit code is considered a failure.
-   `initialDelaySeconds`: The number of seconds to wait after the container has started before the first probe is initiated.
-   `periodSeconds`: The frequency (in seconds) at which the probe is performed.

## Cliffhanger: The `apply` Fails
As shown in the video, attempting to apply this manifest might fail for various reasons, such as a pre-existing pod with the same name or other cluster issues. This sets the stage for troubleshooting in Part 2.

In the next part, we will diagnose the failure and successfully apply the manifest to create the pod with the functioning `livenessProbe`.

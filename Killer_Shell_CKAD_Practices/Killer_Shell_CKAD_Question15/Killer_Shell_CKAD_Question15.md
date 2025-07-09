# CKAD Practice Question 15: Debugging and Fixing a Deployment with a ConfigMap

## Question
Team Moonpie has a `nginx` server `Deployment` called `web-moon` in `Namespace` `moon`. Someone started configuring it but it was never completed. To complete please create a `ConfigMap` called `configmap-web-moon-html` containing the content of file `/opt/course/15/web-moon.html` under the data key-name `index.html`.

The `Deployment` `web-moon` is already configured to work with this `ConfigMap` and serve its content. Test the `nginx` configuration for example using `curl` from a temporary `nginx:alpine` `Pod`.

## Solution

### Step 1: Investigate the Failing Deployment
First, inspect the resources in the `moon` namespace to understand the problem. You'll notice the pods are stuck in a `ContainerCreating` state.

```bash
kubectl get all -n moon
# NAME                            READY   STATUS              RESTARTS   AGE
# pod/web-moon-5f8c7f6f8f-xxxxx   0/1     ContainerCreating   0          60s
# ... (other pods)
```

Describe one of the failing pods to find the root cause. The events section will show that the pod cannot start because the `ConfigMap` it's trying to mount does not exist.

```bash
kubectl describe pod <pod-name> -n moon
# ...
# Events:
#   Type     Reason       Age        From               Message
#   ----     ------       ----       ----               -------
#   Warning  FailedMount  2m         kubelet            MountVolume.SetUp failed for volume "html-volume": configmap "configmap-web-moon-html" not found
```

### Step 2: Create the Missing ConfigMap
Create the `ConfigMap` using the file `/opt/course/15/web-moon.html` and specifying `index.html` as the key.

```bash
kubectl create configmap configmap-web-moon-html --from-file=index.html=/opt/course/15/web-moon.html -n moon
```

Verify the `ConfigMap` was created correctly:
```bash
kubectl describe configmap configmap-web-moon-html -n moon
# Name:         configmap-web-moon-html
# Namespace:    moon
# ...
# Data
# ====
# index.html:
# ----
# <html>
# <body>
# <h1>Web-Moon Web Page</h1>
# <p>This is some great content!</p>
# </body>
# </html>
```

### Step 3: Restart the Deployment
Even after the `ConfigMap` is created, the existing pods will not recover automatically. You must trigger a rollout to force the `Deployment` to create new pods that can successfully mount the `ConfigMap`.

```bash
kubectl rollout restart deployment web-moon -n moon
```

Now, watch the pods. The old ones will terminate, and the new ones will start and enter the `Running` state.
```bash
kubectl get pods -n moon
# NAME                          READY   STATUS    RESTARTS   AGE
# web-moon-7d5c8f8f6b-yyyyy   1/1     Running   0          30s
# ... (other running pods)
```

### Step 4: Test the Web Server
Finally, test that the `nginx` server is serving the content from the `ConfigMap`.

```bash
# Get the IP address of one of the running pods
POD_IP=$(kubectl get pods -n moon -l app=web-moon -o jsonpath='{.items[0].status.podIP}')

# Run a temporary pod and use curl to access the web server
kubectl run test-curl --image=nginx:alpine -n moon --rm -it -- sh -c "apk add --no-cache curl && curl http://$POD_IP"

# Expected output:
# <html>
# <body>
# <h1>Web-Moon Web Page</h1>
# <p>This is some great content!</p>
# </body>
# </html>
```

## Explanation

### Key Concepts
1.  **Debugging Pods**: When a pod is stuck in a non-Running state (like `Pending` or `ContainerCreating`), `kubectl describe pod <pod-name>` is the most important command. The `Events` section at the bottom almost always tells you the exact reason for the failure.
2.  **ConfigMap from File with a Key**: The `--from-file=<key>=<source-file>` syntax is crucial when the desired key name inside the `ConfigMap` is different from the source filename.
3.  **Updating Deployments**: When a `Deployment`'s pods depend on a `ConfigMap` or `Secret` mounted as a volume, creating or updating that resource does not automatically fix existing pods. You must trigger a new rollout with `kubectl rollout restart` so that new pods are created with the updated configuration.

### Exam Tips
-   **Start by Describing**: If a question involves a broken resource, your first step should always be to `describe` it to find the error message.
-   **Imperative Commands are Key**: For speed, use imperative commands like `kubectl create configmap --from-file` instead of writing YAML from scratch.
-   **Remember `rollout restart`**: This is a common pattern in the CKAD exam. If you fix a `ConfigMap` or `Secret` for a `Deployment`, you almost always need to follow it up with a `rollout restart`.
-   **Temporary Test Pods**: Be comfortable with the `kubectl run <name> --image=... --rm -it -- <command>` pattern for testing network connectivity from within the cluster.

## Related Commands
```bash
# Get all resources in a namespace
kubectl get all -n <namespace>

# Describe a pod to find errors
kubectl describe pod <pod-name> -n <namespace>

# Create a ConfigMap from a file with a specific key
kubectl create configmap <map-name> --from-file=<key>=<path-to-file> -n <namespace>

# Force a new rollout for a deployment
kubectl rollout restart deployment <deployment-name> -n <namespace>

# Get the status of a rollout
kubectl rollout status deployment <deployment-name> -n <namespace>
```

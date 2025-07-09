# CKAD Practice Question 14: Managing Secrets in Pods

## Question
You need to make changes on an existing `Pod` in `Namespace` `moon` called `secret-handler`. Create a new `Secret` `secret1` which contains `user=test` and `pass=pwd`. The `Secret`'s content should be available in `Pod` `secret-handler` as environment variables `SECRET1_USER` and `SECRET1_PASS`. The `yaml` for `Pod` `secret-handler` is available at `/opt/course/14/secret-handler.yaml`.

There is existing `yaml` for another `Secret` at `/opt/course/14/secret2.yaml`, create this `Secret` and mount it inside the same `Pod` at `/tmp/secret2`. Your changes should be saved under `/opt/course/14/secret-handler-new.yaml`. Both `Secrets` should only be available in `Namespace` `moon`.

## Solution

### Step 1: Create `secret1`
First, create the `secret1` imperatively using `kubectl` with the specified key-value pairs.

```bash
kubectl create secret generic secret1 --from-literal=user=test --from-literal=pass=pwd -n moon
```

Verify the secret was created and contains the correct data (optional but recommended):
```bash
# Check the secret exists
kubectl get secret secret1 -n moon

# Describe the secret to see keys
kubectl describe secret secret1 -n moon
# Name:         secret1
# Namespace:    moon
# Labels:       <none>
# Annotations:  <none>
#
# Type:  Opaque
#
# Data
# ====
# pass:  3 bytes
# user:  4 bytes
```

### Step 2: Create `secret2`
The YAML for `secret2` is provided. Create it using `kubectl apply`.

```bash
kubectl apply -f /opt/course/14/secret2.yaml -n moon
```

Verify `secret2` was also created:
```bash
kubectl get secret secret2 -n moon
```

### Step 3: Modify the Pod Definition
Copy the existing pod definition to a new file and then modify it. This is a best practice to avoid altering original files.

```bash
cp /opt/course/14/secret-handler.yaml /opt/course/14/secret-handler-new.yaml
```

Now, edit `/opt/course/14/secret-handler-new.yaml` to add the secrets.

**1. Add Environment Variables from `secret1`:**
Add the `env` section to the container spec to pull values from `secret1`.

**2. Add Volume Mount for `secret2`:**
First, define a `volume` that references `secret2`. Then, mount that volume into the container at `/tmp/secret2` using `volumeMounts`.

The final `/opt/course/14/secret-handler-new.yaml` should look like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-handler
  namespace: moon
spec:
  containers:
  - name: secret-handler-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    env:
    - name: SECRET1_USER
      valueFrom:
        secretKeyRef:
          name: secret1
          key: user
    - name: SECRET1_PASS
      valueFrom:
        secretKeyRef:
          name: secret1
          key: pass
    volumeMounts:
    - name: secret-volume-2
      mountPath: /tmp/secret2
  volumes:
  - name: secret-volume-2
    secret:
      secretName: secret2
```

### Step 4: Replace the Pod
Since the pod `secret-handler` already exists, you must use `kubectl replace --force` to delete the old one and create the new one based on your modified YAML.

```bash
kubectl replace -f /opt/course/14/secret-handler-new.yaml --force --grace-period=0
```

### Step 5: Verify the Changes
1.  **Check Environment Variables**: `exec` into the pod and print the environment variables.
    ```bash
    kubectl exec -it secret-handler -n moon -- printenv | grep SECRET1
    # SECRET1_USER=test
    # SECRET1_PASS=pwd
    ```

2.  **Check Volume Mount**: Check the contents of the mounted secret.
    ```bash
    kubectl exec -it secret-handler -n moon -- ls /tmp/secret2
    # You should see the key from secret2 as a file.
    
    kubectl exec -it secret-handler -n moon -- cat /tmp/secret2/<key-file-name>
    # This will display the value from secret2.
    ```

## Explanation

### Key Concepts
1.  **Secrets**: Kubernetes `Secrets` let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. Storing this information in a `Secret` is safer and more flexible than putting it verbatim in a `Pod` definition or in a container image.
2.  **Consuming Secrets as Environment Variables**: You can expose `Secret` data as environment variables in a container. The `valueFrom.secretKeyRef` field allows you to select a specific key from a `Secret` to be the value of an environment variable.
3.  **Consuming Secrets as Volumes**: You can mount a `Secret` as a read-only volume. Each key in the `Secret` becomes a file in the specified `mountPath`. This is useful for configuration files or when you have multiple pieces of data that an application expects to read from the filesystem.
4.  **`kubectl replace`**: This command is used to update an existing resource from a file. The `--force` flag performs an immediate, ungraceful deletion, which is useful in exam scenarios to speed things up.

### Exam Tips
-   **Imperative Commands are Fast**: For creating simple secrets, `kubectl create secret generic --from-literal=...` is much faster than writing a YAML file.
-   **Know Both Ways to Use Secrets**: Be comfortable with consuming secrets as both environment variables and mounted volumes, as questions can test either method.
-   **Modify, Don't Recreate**: When asked to modify an existing resource, always copy its YAML definition first (`cp original.yaml new.yaml`), then modify the copy. Use `kubectl replace` or `kubectl apply` on the new file.
-   **Verification is Key**: Always `exec` into the pod to verify your changes. Use `printenv` for environment variables and `ls`/`cat` for mounted volumes.

## Related Commands
```bash
# Create a generic secret from literal values
kubectl create secret generic <secret-name> --from-literal=<key>=<value> -n <namespace>

# Get a list of secrets
kubectl get secrets -n <namespace>

# Describe a secret to see its keys and type
kubectl describe secret <secret-name> -n <namespace>

# Replace an existing resource from a file
kubectl replace -f <filename.yaml> --force

# Execute a command in a running pod
kubectl exec -it <pod-name> -n <namespace> -- <command>
```

# 15.4 Using Secrets

This section provides a solution for the sample exam assignment on creating and using secrets in a deployment.

## Task

1.  Create a secret that defines the environment variable `password=secret`.
2.  Create a deployment named `secret-app` that starts the `nginx` image.
3.  Use the secret to expose the `password` variable to the pods in the deployment.

---

## Solution Walkthrough

This is a straightforward task that can be accomplished entirely from the command line.

### 1. Create the Secret

First, create the secret using the `kubectl create secret generic` command with the `--from-literal` flag.

```bash
kubectl create secret generic secret-pw --from-literal=password=secret
```

Verify its creation and check the contents (the value will be base64 encoded).

```bash
kubectl describe secret secret-pw
```

### 2. Create the Deployment

Next, create the deployment.

```bash
kubectl create deploy secret-app --image=nginx
```

### 3. Expose the Secret as an Environment Variable

There is no direct flag to add a secret as an environment variable during deployment creation. The best approach is to create the deployment first and then use `kubectl set env` to inject the secret.

```bash
kubectl set env deployment/secret-app --from=secret/secret-pw
```

This command tells Kubernetes to take all key-value pairs from the `secret-pw` secret and set them as environment variables in the containers of the `secret-app` deployment.

### 4. Verify the Solution

Check that the pod is running.

```bash
kubectl get pods
```

Once the pod is in the `Running` state, use `kubectl exec` to print the environment variables inside the container and confirm that the `password` variable is present.

```bash
# Replace <pod-name> with the actual name of your pod
kubectl exec <pod-name> -- env
```

You should see `password=secret` in the output, which confirms the solution is correct.

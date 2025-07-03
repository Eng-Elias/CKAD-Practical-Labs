# 15.10 Exposing Applications

This section provides a solution for the sample exam assignment on exposing a deployment externally using a NodePort service and an Ingress resource.

## Task

1.  Create all resources in the `ckad-ns6` namespace.
2.  Create a deployment named `nginx-deployment` that runs 3 replicas of the `nginx:1.19` image.
3.  Expose the deployment with a Service so that it can be reached on port `32000` on the cluster nodes.
4.  Configure an Ingress resource to access the application at the hostname `mynginx.info`.

---

## Solution Walkthrough

This task involves a multi-step process of creating a deployment, a service, and an ingress.

### Exam Tip: What You Don't Need to Do

Two things you typically **don't** have to do on the real CKAD exam:
*   **Edit `/etc/hosts`:** DNS is considered external to the cluster and should be pre-configured.
*   **Enable Ingress Controller:** The cluster should come with an Ingress controller already enabled (`minikube addons list`) (`minikube addons enable ingress`).

### 1. Create the Namespace

Always start by creating the required namespace. Forgetting this is a common source of errors.

```bash
kubectl create ns ckad-ns6
```

### 2. Create the Deployment

Use `kubectl create deploy` to quickly create the deployment. Remember to specify the namespace!

```bash
kubectl create deploy nginx-deployment --image=nginx:1.19 --replicas=3 -n ckad-ns6
```

### 3. Expose the Deployment with a NodePort Service

#### A. Expose with a ClusterIP Service First

The easiest way to create a service is with `kubectl expose`. This will create a `ClusterIP` service by default.

```bash
kubectl expose deployment nginx-deployment --port=80 -n ckad-ns6
```

#### B. Edit the Service to Change Type and Add NodePort

Now, use `kubectl edit` to change the service type to `NodePort` and specify the port. A common error is using `nodeport` (lowercase) instead of `NodePort` (camel case).

```bash
kubectl edit svc nginx-deployment -n ckad-ns6
```

In the editor, make two changes:
1.  Change `spec.type` from `ClusterIP` to `NodePort`.
2.  Add the `nodePort: 32000` field to the `spec.ports` section.

After saving, you can verify with `kubectl get svc -n ckad-ns6`.

### 4. Create the Ingress Resource

Use `kubectl create ingress` to define the routing rule. It's helpful to use `kubectl create ingress -h` to see examples.

```bash
# The --class flag is often optional, 'default' or 'nginx' are common values
kubectl create ingress nginx-deploy --rule="mynginx.info/=nginx-deployment:80" -n ckad-ns6
```

**Common Mistake:** don't forget the `-n ckad-ns6` flag initially. This creates the Ingress in the `default` namespace. The fix is to delete the wrong Ingress and recreate it in the correct namespace.

```bash
# If you make a mistake:
kubeclt delete ingress nginx-deploy
# Then recreate with the -n ckad-ns6 flag
```

### 5. Verify the Solution

First, check that the Ingress was created correctly.

```bash
kubectl get ingress -n ckad-ns6
```

Then, use `curl` to test the `mynginx.info` hostname. In a correctly configured environment (with DNS or `/etc/hosts` set up), this should return the NGINX welcome page.

```bash
curl mynginx.info
# Expected output: Welcome to nginx!
```

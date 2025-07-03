# 15.8 Using Probes

This section provides a solution for the sample exam assignment on configuring and using readiness probes.

## Task

1.  Create a Pod that runs an `nginx` web server.
2.  The web server should offer its services on port 80.
3.  The Pod must run in the `ckad-ns3` namespace.
4.  Before the main container starts, the Pod must use a **readiness probe** to check that the `/healthz` path on the API server is responding successfully.

---

## Solution Walkthrough

This task requires investigating an API endpoint, finding its address, correctly configuring a probe, and debugging common errors.

### 1. Investigate the API Server `/healthz` Endpoint

The first step is to figure out what command the probe needs to run.

#### A. Find the API Server Address and Port

The API server address is not `localhost`. In a Minikube environment, you can find it with `minikube ip` (which gives an address like `192.168.49.2`) and the port is typically `8443`.

Alternatively, you can find the API server's pod in the `kube-system` namespace and describe it to find its advertised address and port.

```bash
# Find the API server pod
kubectl get pods -n kube-system

# Describe the pod to find the details
kubectl describe pod <kube-apiserver-pod-name> -n kube-system
```

#### B. Test the Endpoint with `curl`

Construct a `curl` command to test the endpoint. The `-k` (or `--insecure`) flag is required because the API server uses a self-signed certificate.

```bash
curl -k https://192.168.49.2:8443/healthz
# Expected output: ok
```

A successful command returns `ok`. This is the exact command we will use in our readiness probe.

### 2. Assemble and Debug the Initial YAML

Start by creating a YAML file (`task15-8.yaml`) for the pod.

#### A. Find a Template

Copy a basic probe example from the [official documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/). It's fine to start with a `livenessProbe` and adapt it.

#### B. Create the Initial Manifest

Modify the template:
*   Change `livenessProbe` to `readinessProbe`.
*   Set the `exec.command` to the `curl` command we tested earlier.
*   Use a basic `nginx` container.

Here is an example of the first attempt, which contains an error:

```yaml
# task15-8.yaml (Initial broken version)
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness-exec
  name: liveness-exec
spec:
  containers:
  - name: nginx
    image: nginx
    readinessProbe:
      exec:
        command:
        - curl
        - k # <-- This is a mistake! It should be -k
        - https://192.168.49.2:8443/healthz # <-- your minikube ip and port
      initialDelaySeconds: 5
      periodSeconds: 5
```

### 3. Diagnose the Failing Probe

Create the pod and observe its status.

```bash
kubectl create -f task15-8.yaml
kubectl get pods -w
# NAME            READY   STATUS    RESTARTS   AGE
# liveness-exec   0/1     Running   0          20s
```

The Pod gets stuck in the `0/1` ready state, which means the readiness probe is failing. Use `kubectl describe` to find out why.

```bash
kubectl describe pod liveness-exec
```

In the `Events` section, you'll see a `Warning` with a message like `Readiness probe failed: ... context deadline exceeded`. Looking at the probe definition in the describe output reveals the typo: `curl, k, ...`.

### 4. Fix the Probe and Add Final Requirements

Now, correct the YAML to fix the probe and add the remaining requirements from the task.

#### A. Fix the Probe Command

Change `k` to `-k` in the `command` array.

#### B. Add Namespace and Port

*   Add `namespace: ckad-ns3` to the metadata.
*   Add the `ports` section to the container spec. Use `kubectl explain pod.spec.containers.ports` to find the correct syntax if you are unsure. It requires a list format.

### 5. Final YAML and Verification

Here is the complete, correct YAML:

```yaml
# task15-8.yaml (Final correct version)
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
  namespace: ckad-ns3
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      exec:
        command:
        - curl
        - "-k"
        - https://192.168.49.2:8443/healthz # <-- your minikube ip and port
      initialDelaySeconds: 5
      periodSeconds: 5
```

Create the resources:

```bash
# Create the namespace first
kubectl create ns ckad-ns3

# Create the pod from the fixed file
kubectl create -f task15-8.yaml
```

Monitor the pod's status. It should become `1/1` ready after the initial delay, confirming the probe is successful.

```bash
kubectl get pods -n ckad-ns3 -w
# NAME            READY   STATUS    RESTARTS   AGE
# liveness-exec   0/1     Running   0          5s
```


```bash
kubectl get pods -n ckad-ns3 -w
# NAME            READY   STATUS    RESTARTS   AGE
# liveness-exec   1/1     Running   0          10s
```
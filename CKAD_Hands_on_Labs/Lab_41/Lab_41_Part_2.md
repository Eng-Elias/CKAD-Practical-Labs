# CKAD Lab 41 (Part 2): Exposing a Deployment with a ClusterIP Service

## Objective
With the `deploy-foo` Deployment running, the next step is to expose it internally so other applications within the cluster can reliably access it. We will do this by creating a `ClusterIP` Service.

## The Challenge

-   Expose the `deploy-foo` Deployment.
-   The Service should be named `svc-foo`.
-   It should be a `ClusterIP` type service (which is the default).
-   It should listen on port `6379` and forward traffic to the pods' target port `6379`.

## Part 2 Solution: The Service
A Service provides a stable endpoint (a consistent IP address and DNS name) for a set of pods. The `kubectl expose` command is the fastest way to create one.

### Step 1: The Imperative Command
We can expose the deployment and name the service in a single command.

**Command:**
```bash
kubectl expose deployment deploy-foo --port=6379 --target-port=6379 --name=svc-foo
```

**Command Breakdown:**
-   `kubectl expose deployment deploy-foo`: Targets the deployment we want to expose.
-   `--port=6379`: The port that the Service itself will listen on.
-   `--target-port=6379`: The port on the backend pods that the service will forward traffic to.
-   `--name=svc-foo`: **Crucial Step:** Gives the service a unique name. If omitted, the service would be named `deploy-foo`, which can be confusing.

### Step 2: How Does it Work? The Power of Selectors
When you run `kubectl expose`, Kubernetes automatically does two things:
1.  It inspects the `deploy-foo` deployment to find its pod labels (in our case, `app=foo`).
2.  It creates the `svc-foo` Service with a **selector** that matches those labels (`selector: app=foo`).

This selector continuously scans for pods with the `app=foo` label and automatically adds their IP addresses to the service's list of endpoints. This is the core mechanism that connects a service to its backend pods.

### Step 3: Verify the Service
Check that the service was created and that its selector is correct.

**Commands:**
```bash
# Get a summary of the service
kubectl get service svc-foo

# Describe the service to see details, including the selector
kubectl describe service svc-foo
```

**Expected Output for `describe`:**
```
Name:              svc-foo
Namespace:         default
Labels:            app=foo
Annotations:       <none>
Selector:          app=foo
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.101.123.45
IPs:               10.101.123.45
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.244.0.5:6379,10.244.0.6:6379,10.244.0.7:6379
Session Affinity:  None
Events:            <none>
```
Notice the `Selector: app=foo` and the `Endpoints` list, which shows the IP addresses of the three pods from our deployment.

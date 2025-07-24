# CKAD Lab 44 (Part 1): Exploring the `readinessProbe` with `kubectl explain`

## Objective
This two-part lab covers how to configure a `readinessProbe` for a container. A `readinessProbe` is used by Kubernetes to know when a container is ready to start accepting traffic. If a readiness probe fails, the pod's IP address is removed from the endpoints of all Services that match the pod, effectively taking it out of the load-balancing rotation until it becomes ready again.

## The Challenge
Create a pod named `foo-pod` with an `nginx` container. Configure a `readinessProbe` that performs an HTTP GET request on the path `/` on port `80`.

## The Problem: Finding a Simple Example
As with Persistent Volumes, finding a clean, minimal example for a specific probe type in the official documentation can be difficult. The documentation often presents complex examples that combine multiple features.

When you can't find a simple example, `kubectl explain` is the most reliable tool to build the manifest from scratch.

## Part 1 Solution: Exploring the Probe's Structure
We will use `kubectl explain` to drill down into the pod specification and find the fields for an HTTP-based readiness probe.

### Step 1: Find the `readinessProbe` field
First, let's see where the `readinessProbe` is defined within a container's spec.

**Command:**
```bash
kubectl explain pod.spec.containers.readinessProbe
```

**Partial Output:**
```
KIND:     Pod
VERSION:  v1

RESOURCE: readinessProbe <Object>

DESCRIPTION:
    Periodic probe of container service readiness. Container will be removed from service endpoints if the probe fails.

FIELDS:
   exec <Object>
   httpGet <Object>
   tcpSocket <Object>
   ...and others like initialDelaySeconds, periodSeconds, etc.
```
This output confirms that `readinessProbe` is an object and shows the three types of probes we can define: `exec` (run a command), `httpGet` (make an HTTP request), and `tcpSocket` (check a TCP port). Our requirement is for an `httpGet` probe.

### Step 2: Explore the `httpGet` Probe
Now, let's see what fields are required for the `httpGet` object.

**Command:**
```bash
kubectl explain pod.spec.containers.readinessProbe.httpGet
```

**Output:**
```
KIND:     Pod
VERSION:  v1

RESOURCE: httpGet <Object>

DESCRIPTION:
    HTTPGet specifies the http request to perform.

FIELDS:
   path <string>
   port <string> -required-
   host <string>
   httpHeaders <[]Object>
   scheme <string>
```
This gives us everything we need. The `httpGet` object requires a `port` and allows us to specify a `path`. 

## Conclusion for Part 1
With `kubectl explain`, we have precisely identified the YAML structure needed for our `readinessProbe`. We know that under `readinessProbe`, we need an `httpGet` object, which in turn contains `path` and `port` fields.

In Part 2, we will use this knowledge to construct the final pod manifest.

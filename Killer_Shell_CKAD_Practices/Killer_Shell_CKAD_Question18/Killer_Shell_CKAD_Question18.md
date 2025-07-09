# CKAD Practice Question 18: Troubleshooting a Service with No Endpoints

## Question
There seems to be an issue with the `mars` namespace where the `ClusterIP` service `manager-api-service` should make the pods of a deployment available inside the cluster, but it is not working. You can test it with a `curl` from a temporary `nginx:alpine` pod. Check the misconfiguration and apply a fix.

## Solution

### Step 1: Diagnose the Service
First, attempt to connect to the service from a temporary pod to confirm the issue. The connection will time out.

```bash
# The -m 3 flag sets a 3-second timeout
kubectl run tmp-pod --image=nginx:alpine -n mars --rm -it -- sh -c "apk add --no-cache curl && curl -m 3 http://manager-api-service:4444"
# Command timed out after 3 seconds
```

The next step is to `describe` the service. The most important piece of information is the `Endpoints` field. If it's `<none>`, it means the service has not found any pods matching its selector.

```bash
kubectl describe svc manager-api-service -n mars
# Name:              manager-api-service
# Namespace:         mars
# Labels:            <none>
# Annotations:       <none>
# Selector:          id=manager-api-deployment
# Type:              ClusterIP
# IP Family Policy:  SingleStack
# IP Families:       IPv4
# IP:                10.108.138.14
# IPs:               10.108.138.14
# Port:              <unset>  4444/TCP
# TargetPort:        80/TCP
# Endpoints:         <none>  <-- THIS IS THE PROBLEM
# Session Affinity:  None
# Events:            <none>
```

### Step 2: Find the Label Mismatch
Since there are no endpoints, the service's selector is not matching the labels of any pods. Let's compare the service's selector with the pod labels.

```bash
# Get the service's selector
kubectl get svc manager-api-service -n mars -o jsonpath='{.spec.selector}'
# {"id":"manager-api-deployment"}

# Get the labels of the pods that should be targeted
kubectl get pods -n mars --show-labels
# NAME                                     READY   STATUS    RESTARTS   AGE   LABELS
# manager-api-deployment-5c4f8f6f8f-xxxxx   1/1     Running   0          10m   id=manager-api-pod,pod-template-hash=...
# ... (other pods)
```
The service is looking for pods with the label `id=manager-api-deployment`, but the pods have the label `id=manager-api-pod`. This mismatch is the root cause.

### Step 3: Fix the Service Selector
Edit the service and change the selector to match the pods' label.

```bash
kubectl edit svc manager-api-service -n mars
```

Find the `selector` field and change `manager-api-deployment` to `manager-api-pod`:
```yaml
# ...
spec:
  clusterIP: 10.108.138.14
  ports:
  - port: 4444
    protocol: TCP
    targetPort: 80
  selector:
    id: manager-api-pod # <-- Change this value
# ...
```

### Step 4: Verify the Fix
First, `describe` the service again. The `Endpoints` field should now be populated with the IP addresses of the pods.

```bash
kubectl describe svc manager-api-service -n mars
# ...
# Endpoints:         10.244.0.22:80,10.244.0.23:80,10.244.0.24:80 + 1 more...
# ...
```

Now, run the test again from the temporary pod. It should connect successfully.
```bash
kubectl run tmp-pod --image=nginx:alpine -n mars --rm -it -- sh -c "apk add --no-cache curl && curl http://manager-api-service:4444"
# Welcome to nginx!
# ...
```

## Explanation

### Key Concepts
1.  **Service and Endpoints**: A Kubernetes `Service` provides a stable endpoint (a `ClusterIP` and DNS name) for a set of pods. Kubernetes automatically creates an `Endpoints` object that tracks the IP addresses of the pods that match the service's `selector`.
2.  **Label Selectors**: The link between a `Service` and its `Pods` is the label selector. If the `selector` in the `Service` definition does not match the labels on any `Pods`, the `Endpoints` object will be empty, and the service will not be able to forward traffic.
3.  **Debugging Workflow**: A common workflow for debugging services is to first check if the `Endpoints` are populated. If they are not, the problem is almost always a mismatch between the service's `selector` and the pods' `labels`.

### Exam Tips
-   **`Endpoints: <none>` is the biggest clue**: If you `describe` a service and see no endpoints, immediately check for a label mismatch. This is a very common exam scenario.
-   **Services select Pods, not Deployments**: A frequent mistake is to configure a service's selector to match the labels of a `Deployment` instead of the labels of the `Pods` created by that `Deployment`. Always check the pod labels.
-   **Use `kubectl edit` for quick fixes**: In an exam, `kubectl edit` is the fastest way to correct a single field in a resource's definition.

## Related Commands
```bash
# Describe a service to check its selector and endpoints
kubectl describe service <service-name> -n <namespace>

# Get the labels for all pods in a namespace
kubectl get pods --show-labels -n <namespace>

# Edit a live resource in the cluster
kubectl edit <resource-type> <resource-name> -n <namespace>

# Run a temporary pod for testing
kubectl run <pod-name> --image=<image> -n <namespace> --rm -it -- <command>
```

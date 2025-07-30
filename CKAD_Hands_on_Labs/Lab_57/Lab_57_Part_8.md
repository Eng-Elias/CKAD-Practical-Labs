# CKAD Lab 57 (Part 8): The Final Fix and Verification

## Objective
Apply the final correction to the `Deployment` manifest, successfully create the resource in the cluster, and verify that all requirements from the original challenge have been met.

## Part 8 Solution: The Selector/Label Match

### The Final Error: `selector does not match template labels`
The most critical rule for a `Deployment` (and `ReplicaSet`, `StatefulSet`, etc.) is that the pod selector must match the labels of the pods it is supposed to manage. 

-   `spec.selector.matchLabels`: Tells the `Deployment` controller which pods to look for.
-   `spec.template.metadata.labels`: Assigns labels to the pods that the `Deployment` creates.

These two sets of labels **must match**. The error from Part 7 occurred because they were different.

### The Complete and Corrected `deployment.yaml`
This is the final manifest with all corrections applied, meeting all lab requirements.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
  labels:
    app: redis
spec:
  replicas: 3
  # This selector MUST match the template labels below
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      # These labels are applied to each Pod
      labels:
        app: redis
    spec:
      containers:
      - name: foo-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "100m"
        volumeMounts:
        - name: data-volume
          mountPath: /data
        - name: config-volume
          mountPath: /etc/config
      # Node affinity to schedule pods only on the labeled node
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - redis-node
      # All volumes are defined at the pod level
      volumes:
      - name: data-volume
        emptyDir: {}
      - name: config-volume
        configMap:
          name: cm1
```

### Verification Steps
After applying the final manifest (`kubectl apply -f deployment.yaml`), we verify each requirement.

1.  **Check the Deployment and Pods:**
    ```bash
    # Verify the deployment is created and replicas are available
    kubectl get deployment foo-redis

    # Verify 3 pods are running or creating on the correct node
    # The 'ContainerCreating' status confirms scheduling (nodeAffinity) was successful
    kubectl get pods -o wide
    ```

2.  **Inspect the Pods in Detail:**
    ```bash
    # Pick one of the pod names from the previous command
    POD_NAME=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')

    # Describe the pod to see labels, node, and resource requests
    kubectl describe pod $POD_NAME

    # Exec into the pod to verify the mounts
    kubectl exec -it $POD_NAME -- sh

    # Inside the pod shell, check the mounts:
    # ls /data (should be an empty directory)
    # ls /etc/config (should contain a file named 'key1' with content 'val1')
    # exit
    ```

This completes this comprehensive lab, covering many core CKAD concepts in a single, complex resource.

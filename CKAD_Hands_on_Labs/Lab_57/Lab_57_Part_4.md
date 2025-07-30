# CKAD Lab 57 (Part 4): Adding the Node Affinity Rule

## Objective
Add the `nodeAffinity` rule to the `Deployment` manifest to ensure that all pods created by this deployment are scheduled only on `node01`.

## Part 4 Solution: Building the `nodeAffinity` Block

### Step 1: Add the `affinity` Block to the Pod Template
Using the path discovered in the previous part (`deployment.spec.template.spec.affinity...`), we add the `affinity` block to the pod template spec.

As we have seen in previous labs, the fastest way to build this is to add the fields and then use the API server's validation errors to fix the structure. The key is remembering that both `nodeSelectorTerms` and `matchExpressions` must be arrays (lists in YAML).

### The Modified `deployment.yaml`
Here is the updated manifest with the `nodeAffinity` section included.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
  labels:
    app: redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: foo-container
        image: redis:alpine
        resources:
          requests:
            cpu: "100m"
        ports:
        - containerPort: 6379
      # Added nodeAffinity to schedule pods on the correct node
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - redis-node
```

## Next Steps
With the scheduling constraint in place, the next requirement is to add volumes to the pod. Part 5 will cover how to define and mount an `emptyDir` volume.

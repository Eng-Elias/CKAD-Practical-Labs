# CKAD Lab 57 (Part 3): Configuring Ports and Preparing for Node Affinity

## Objective
Continue modifying the `Deployment` manifest to expose the correct container port and prepare for adding a `nodeAffinity` rule by labeling the target node.

## Part 3 Solution: Ports and Node Labeling

### Step 1: Expose the Container Port
This is a straightforward change to the `ports` section of the container definition.

### Step 2: Prepare for Node Affinity
Before we can add a `nodeAffinity` rule to the pod spec, we must first apply a label to the target node (`node01`) that the rule can match against.

**Command:**
```bash
# Apply a unique label to node01
kubectl label nodes node01 app=redis-node
```

### Step 3: Discover the `nodeAffinity` Path
Using `kubectl explain`, we can again trace the path to the `nodeAffinity` fields within the `Deployment` spec. The path is the same as for a `Pod`, but nested under the `template`.

**The Discovery Path:**
`deployment.spec.template.spec.affinity.nodeAffinity...`

### The Modified `deployment.yaml`
Here is the updated manifest with the correct port and labels. The `nodeAffinity` section is not yet added.

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
        # The correct port for Redis
        ports:
        - containerPort: 6379
```

## Next Steps
With the node labeled and the port correctly configured, Part 4 will focus on adding the `nodeAffinity` block to the manifest to enforce the scheduling constraint.

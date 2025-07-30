# CKAD Lab 57 (Part 2): Adding Resource Requests and Labels

## Objective
Continue building the `Deployment` manifest by adding CPU resource requests and applying the required labels. This part clarifies the distinction between labeling a `Pod` and a `container`.

## Part 2 Solution: Adding CPU Requests and Labels

### Step 1: Add Resource Requests
Using `kubectl explain`, we can find the correct path to specify resource requests for a container.

**The Discovery Path:**
`kubectl explain deployment.spec.template.spec.containers.resources.requests`

This path leads to the `cpu` and `memory` fields. We add the `resources` block to our container definition.

### Step 2: Clarifying Labels
The requirement to "label the container" is a common source of confusion. **Containers themselves cannot be labeled.** Labels are part of an object's `metadata`. In a `Deployment`, this means you apply labels to the Pods created by the `Deployment` by setting the `spec.template.metadata.labels` field.

Crucially, the `Deployment`'s `spec.selector.matchLabels` field must match the labels you apply to the pod template. This is how the `Deployment` knows which pods to manage.

### The Modified `deployment.yaml`
Here is the updated manifest with the new `resources` and `labels` sections.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
  # The deployment object itself can have labels
  labels:
    app: redis
spec:
  replicas: 3
  # The selector MUST match the pod template's labels
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      # These are the labels applied to each Pod
      labels:
        app: redis
    spec:
      containers:
      - name: foo-container
        image: redis:alpine
        # Added resource requests for the container
        resources:
          requests:
            cpu: "100m" # 100 millicpu (0.1 cpu)
        ports:
        - containerPort: 80 # Will be changed later
```

## Next Steps
With resource requests and labels correctly configured, Part 3 will focus on exposing the correct container port.

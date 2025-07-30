# CKAD Lab 57 (Part 5): Mounting an `emptyDir` Volume

## Objective
Add an `emptyDir` volume to the `Deployment`'s pod template. An `emptyDir` volume is a simple, temporary directory that is created when a pod is assigned to a node and exists as long as that pod is running on that node. It is useful for sharing files between containers in the same pod or as a temporary scratch space.

## Part 5 Solution: The Two-Step Volume Mount
Mounting any volume in Kubernetes, including an `emptyDir`, follows a two-step process.

### Step 1: Define the Volume in the Pod Spec
First, you add a `volumes` section to the pod template's spec (`spec.template.spec`). Here, you define the volume, give it a name, and specify its type. For an `emptyDir`, you simply provide an empty object `{}`.

### Step 2: Mount the Volume in the Container
Next, you add a `volumeMounts` section to the container definition. Here, you reference the volume you just created by its `name` and specify the `mountPath` where it should be accessible inside the container.

### The Modified `deployment.yaml`
Here is the updated manifest with the `emptyDir` volume and mount.

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
        # Step 2: Mount the volume into the container
        volumeMounts:
        - name: data-volume
          mountPath: /data
      affinity:
        # ... (affinity rules from Part 4)
      # Step 1: Define the volume at the pod level
      volumes:
      - name: data-volume
        emptyDir: {}
```

## Next Steps
The final requirement is to mount the `ConfigMap` created in Part 1 as another volume. Part 6 will cover how to add a second volume definition and mount it into the container.

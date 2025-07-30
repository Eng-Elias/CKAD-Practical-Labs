# CKAD Lab 57 (Part 6): Mounting a `ConfigMap` as a Volume

## Objective
Add the final required volume to the `Deployment`, mounting the `cm1` `ConfigMap` into the container's filesystem. This demonstrates how to handle multiple volumes in a single pod.

## Part 6 Solution: Adding the `ConfigMap` Volume
The process is identical to adding the `emptyDir` volume: define it at the pod level and mount it at the container level.

### Step 1: Add a `ConfigMap` Volume to the Pod Spec
We add a second item to the `volumes` array in the pod template. This new volume is given a name (e.g., `config-volume`) and its source is specified as a `configMap`, referencing `cm1` by name.

### Advanced: Projecting Specific Keys with `items`
Instead of mounting every key from the `ConfigMap` as a file, you can use the `items` field to select specific keys and control their destination paths within the mounted volume. This is a powerful feature for fine-grained configuration.

### Step 2: Add a Second `volumeMount` to the Container
We add a second item to the `volumeMounts` array in the container definition. This mount references the `config-volume` by name and specifies the mount path `/etc/config`.

### The Modified `deployment.yaml`
Here is the manifest with the `ConfigMap` volume and mount added.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
  # ... (metadata from previous parts)
spec:
  # ... (spec from previous parts)
  template:
    # ... (template metadata from previous parts)
    spec:
      containers:
      - name: foo-container
        image: redis:alpine
        # ... (resources and ports from previous parts)
        volumeMounts:
        # The first volume mount for emptyDir
        - name: data-volume
          mountPath: /data
        # The second volume mount for the ConfigMap
        - name: config-volume
          mountPath: /etc/config
      affinity:
        # ... (affinity rules from Part 4)
      volumes:
      # The first volume definition for emptyDir
      - name: data-volume
        emptyDir: {}
      # The second volume definition for the ConfigMap
      - name: config-volume
        configMap:
          # Name of the ConfigMap to mount
          name: cm1
```

## Next Steps
With all the requirements now translated into the YAML manifest, the `Deployment` is complete. Part 7 will cover applying the final manifest, verifying that all components are working as expected, and troubleshooting any potential issues.

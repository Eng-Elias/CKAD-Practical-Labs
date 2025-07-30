# CKAD Lab 57 (Part 1): A Complex Deployment - Initial Setup

## Objective
This is a comprehensive, multi-part lab that involves creating a single `Deployment` with a wide array of requirements, touching on configuration, storage, resource management, and scheduling. 

## The Full Challenge
1.  Create a `ConfigMap` named `cm1` with the data `key1=val1`.
2.  Create a `Deployment` named `foo-redis` with the following specifications:
    -   Image: `redis:alpine`
    -   Replicas: `3`
    -   Resource Requests: `100m` (0.1) CPU.
    -   Labels: `app=redis`.
    -   Exposes container port `6379`.
    -   Must be scheduled on `node01`.
    -   Mounts an `emptyDir` volume at `/data`.
    -   Mounts the `cm1` `ConfigMap` as a volume at `/etc/config`.

## Part 1 Solution: Prerequisites and Basic Manifest
This part covers creating the necessary `ConfigMap` and building the initial `Deployment` manifest.

### Step 1: Create the ConfigMap
First, we create the `ConfigMap` that the deployment will later mount.

**Command:**
```bash
kubectl create configmap cm1 --from-literal=key1=val1
```

### Step 2: Generate the Base Deployment Manifest
We start by getting a standard `Deployment` YAML from the Kubernetes documentation. This provides a solid foundation to build upon.

### Step 3: Initial Modifications
We edit the base manifest to meet the first set of requirements: name, image, and replica count.

**`deployment.yaml` (In Progress):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-redis
  labels:
    app: nginx # We will change this later
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx # This must also be changed
  template:
    metadata:
      labels:
        app: nginx # And this too
    spec:
      containers:
      - name: foo-container # A suitable name
        image: redis:alpine
        ports:
        - containerPort: 80 # This will be changed
```

## Next Steps
With the basic structure in place, Part 2 will focus on using `kubectl explain` to find the correct fields for adding CPU resource requests to the container specification.

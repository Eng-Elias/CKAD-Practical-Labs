# CKAD Lab 58 (Part 2): Uncovering the Real Scheduling Constraint and a Missing ConfigMap

## Objective
With the pods still `Pending` after the first fix, we must dig deeper to find the true cause of the scheduling failure and then resolve the next error that appears.

## Part 2 Solution: From `Pending` to `ContainerCreating`

### Step 1: Inspect the Full Manifest
The `kubectl describe` command provides a summary, but it can hide details like `affinity` rules. To see the complete definition of a resource, you must inspect its full YAML.

```bash
kubectl get deployment foo-redis -o yaml
```

Inspecting the YAML reveals the true problem: a `nodeAffinity` rule requiring a different label.

```yaml
# Excerpt from the deployment's YAML
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: foo
                operator: In
                values:
                - bar
```

### Step 2: Apply the Correct Node Label
The real scheduling constraint was the `nodeAffinity` rule, not the `nodeSelector`. We apply the correct label.

```bash
kubectl label node node01 foo=bar
```

### Step 3: A New Problem - `ContainerCreating`
With the correct label applied, the pods are now scheduled, but they get stuck in the `ContainerCreating` state. This is progress! It means we solved the scheduling issue and have moved on to a new problem.

We describe the pod again:

```bash
kubectl describe pod <pod-name>
```

**New Events:**
```
Type     Reason       Age   From               Message
----     ------       ----  ----               -------
Warning  FailedMount  2m    kubelet            MountVolume.SetUp failed for volume "cm1": configmap "cm1" not found
```

The error is clear: the pod is trying to mount a `ConfigMap` named `cm1`, but it doesn't exist.

### Step 4: Create the Missing ConfigMap
We create the required `ConfigMap`.

```bash
kubectl create configmap cm1 --from-literal=data=hello
```

## The Second Cliffhanger
Even after creating the `ConfigMap`, the pods remain stuck in `ContainerCreating`. The `describe` events now show a timeout error related to the volume mount. This tells us there is *still* another problem with the pod's volumes. Part 3 will investigate why the `ConfigMap` volume mount is still failing.

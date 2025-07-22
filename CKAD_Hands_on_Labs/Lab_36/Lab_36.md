# CKAD Lab 36: Changing an Environment Variable in a Running Pod

## Objective
Update the value of an environment variable in a running pod. This lab highlights the immutable nature of certain pod specifications.

## The Challenge
-   A pod named `foo-pod` is running.
-   It has an environment variable named `XYZ` with the value `hello`.
-   Your task is to change the value of the `XYZ` environment variable to `foo-bar`.

## The Trick: Pods are (Mostly) Immutable
You cannot directly change the environment variables, image, or other core specifications of a running pod. The correct procedure is to export the pod's definition, modify it, and then replace the old pod with a new one based on the updated definition.

## Solution: The "Get, Edit, Delete, Apply" Workflow

### Step 1: Inspect the Running Pod
First, use `kubectl describe` to confirm the current environment variable's value.

**Command:**
```bash
kubectl describe pod foo-pod
```
In the `Environment` section of the output, you will see `XYZ: hello`.

### Step 2: Get the Pod's YAML Definition
Export the running pod's configuration into a local YAML file.

**Command:**
```bash
kubectl get pod foo-pod -o yaml > pod-definition.yaml
```

### Step 3: Edit the YAML File
Open `pod-definition.yaml` in an editor and find the `env` section. Change the value of the `XYZ` variable.

**Before:**
```yaml
    env:
    - name: XYZ
      value: hello
```

**After:**
```yaml
    env:
    - name: XYZ
      value: foo-bar
```

**Note:** You can leave the system-generated fields like `uid`, `resourceVersion`, and the entire `status` block in the file. `kubectl apply` is intelligent enough to ignore them during creation.

### Step 4: Delete the Old Pod
Before you can create the new one, you must delete the existing pod.

**Command:**
```bash
kubectl delete pod foo-pod
```

### Step 5: Apply the Modified YAML
Now, create the new pod using your updated manifest.

**Command:**
```bash
kubectl apply -f pod-definition.yaml
```

### Step 6: Verify the Change
Finally, describe the new pod to confirm that the environment variable has been updated.

**Command:**
```bash
kubectl describe pod foo-pod
```
This time, the `Environment` section will show `XYZ: foo-bar`, confirming the update was successful.

## Exam Tip
This workflow applies to any immutable field in a pod's spec. If you are asked to change something on a running pod and `kubectl edit` or `kubectl set` doesn't work, remember this "get, edit, delete, apply" pattern. It's a reliable method for making such changes.

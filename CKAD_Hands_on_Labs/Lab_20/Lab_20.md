# CKAD Lab 20: Overriding Labels on a Running Pod

## Objective
Learn how to add and, more importantly, override labels on a running Kubernetes pod without needing to restart or recreate it. This is a fundamental skill for dynamically managing and selecting resources.

## What are Labels?
Labels are key-value pairs that are attached to Kubernetes objects, such as pods. They are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users but do not directly imply semantics to the core system.

Labels are the primary way to organize and select subsets of objects. For example, a Service uses a `selector` based on labels to determine which pods should receive traffic.

## Key Command: `kubectl label --overwrite`
The `kubectl label` command is used to add or update labels on any Kubernetes resource. By default, it will error out if you try to change a label that already exists.

To force an update to an existing label, you must use the `--overwrite` flag.

## Solution
This lab walks through creating a pod, adding a label, and then overriding that label.

### Step 1: Create a Pod
First, create a simple `nginx` pod. This pod will be created with a default label.

**Command:**
```bash
kubectl run nginx --image=nginx
```

### Step 2: View Initial Labels
Use the `--show-labels` flag to see the labels currently on the pod.

**Command:**
```bash
kubectl get pod nginx --show-labels
```

**Expected Output:**
```
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          30s   run=nginx
```
As you can see, `kubectl run` automatically added the label `run=nginx`.

### Step 3: Add a New Label
Now, let's add a new label, `env=dev`, to the running pod.

**Command:**
```bash
kubectl label pod nginx env=dev
```

**Verification:**
```bash
kubectl get pod nginx --show-labels
```

**Expected Output:**
```
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          1m    env=dev,run=nginx
```

### Step 4: Override an Existing Label
Imagine we need to promote this application to production. We need to change the `env` label from `dev` to `prod`. If you try to do this without the `--overwrite` flag, you will get an error.

**Correct Command with `--overwrite`:**
```bash
kubectl label pod nginx env=prod --overwrite
```

### Step 5: Verify the Final Labels
Check the labels one last time to confirm the override was successful.

**Command:**
```bash
kubectl get pod nginx --show-labels
```

**Expected Output:**
```
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          2m    env=prod,run=nginx
```

The `env` label has been successfully changed from `dev` to `prod`.

## Exam Tip
In the CKAD exam, you will frequently be asked to modify existing resources. Knowing how to use `kubectl label --overwrite` is much faster than editing the resource's YAML (`kubectl edit`). Practice this command to save valuable time.

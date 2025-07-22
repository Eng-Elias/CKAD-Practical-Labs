# CKAD Lab 39: Creating a ConfigMap with Multiple Key-Value Pairs

## Objective
Create a Kubernetes `ConfigMap` with multiple key-value pairs using a single imperative command. `ConfigMaps` are used to store non-confidential data in key-value pairs, allowing you to decouple configuration from your application code.

## The Challenge
Create a `ConfigMap` with the following specifications:
-   **Name:** `cm-foo`
-   **Data:**
    -   `foo` = `bar`
    -   `foo2` = `bar2`
    -   `foo3` = `bar3`

## Solution Strategy
Similar to creating secrets, the fastest way to create a `ConfigMap` from literal values is with the `kubectl create configmap` command, using the `--from-literal` flag for each key-value pair.

### Step 1: The Imperative Command
Construct a single command that specifies the `ConfigMap`'s name and provides each key-value pair.

**Command:**
```bash
kubectl create configmap cm-foo \
  --from-literal=foo=bar \
  --from-literal=foo2=bar2 \
  --from-literal=foo3=bar3
```

**Command Breakdown:**
-   `kubectl create configmap cm-foo`: Specifies that we are creating a `ConfigMap` named `cm-foo`.
-   `--from-literal=key=value`: This flag is repeated for each key-value pair you want to include in the `ConfigMap`.

### Step 2: Verify the ConfigMap
After creating the `ConfigMap`, you can verify its existence and inspect its data.

**Commands:**
```bash
# Check that the ConfigMap was created
kubectl get configmap cm-foo

# Describe the ConfigMap to see its data
kubectl describe configmap cm-foo
```

**Expected Output for `describe`:**
```
Name:         cm-foo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
foo:
----
bar
foo2:
-----
bar2
foo3:
-----
bar3
Events:  <none>
```
Unlike `Secrets`, the `describe` command for `ConfigMaps` shows the values in plain text, as they are not considered confidential.

## Exam Tip
For the CKAD exam, using `kubectl create configmap --from-literal` is the most efficient way to create `ConfigMaps` from simple key-value data. It saves you from writing a full YAML file, which is a significant time-saver. Practice this command until it's second nature.

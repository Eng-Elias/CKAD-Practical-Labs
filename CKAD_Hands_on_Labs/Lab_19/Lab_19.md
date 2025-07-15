# CKAD Lab 19: Creating a ConfigMap from Literal Values

## Objective
Learn how to create a Kubernetes ConfigMap imperatively using the `--from-literal` flag. This method allows you to define configuration data directly on the command line, which is extremely fast and useful during the CKAD exam.

## What is a ConfigMap?
A ConfigMap is a Kubernetes object used to store non-confidential configuration data in key-value pairs. By decoupling configuration from your application's container image, you make your applications more portable and easier to manage across different environments (e.g., development, staging, production).

Pods can consume ConfigMaps as:
-   Environment variables.
-   Command-line arguments.
-   Configuration files mounted in a volume.

## Key Command: `kubectl create configmap --from-literal`
The `kubectl create configmap` command is used to create ConfigMaps. The `--from-literal` flag allows you to specify a key-value pair directly. You can use this flag multiple times to include several key-value pairs in a single ConfigMap.

## Solution
This lab demonstrates how to create ConfigMaps with single and multiple data items and how to inspect them.

### Step 1: Create a ConfigMap with Multiple Key-Value Pairs
Use the `--from-literal` flag twice to create a ConfigMap named `my-config` with two data items.

**Command:**
```bash
kubectl create configmap my-config --from-literal=app.name=my-app --from-literal=app.version=v1.2.3
```

### Step 2: Inspect the ConfigMap
You can use `kubectl get` and `kubectl describe` to view the ConfigMap and its data.

**Commands:**
```bash
# List all ConfigMaps
kubectl get configmap
# or use the short name 'cm'
kubectl get cm

# Describe the new ConfigMap to see its key-value data
kubectl describe configmap my-config
```

**Expected `describe` Output:**
```
Name:         my-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
app.name:
----
my-app
app.version:
----
v1.2.3

BinaryData
====

Events:  <none>
```

### Step 3: Create a ConfigMap with a Single Key-Value Pair
Let's create another ConfigMap, this time with only one key-value pair.

**Command:**
```bash
kubectl create configmap another-config --from-literal=log_level=debug
```

### Step 4: View ConfigMap Data in YAML or JSON
For detailed inspection or to generate a manifest, you can output the ConfigMap's definition in YAML or JSON format.

**Command:**
```bash
# Get the 'another-config' ConfigMap in YAML format
kubectl get cm another-config -o yaml
```

**Expected YAML Output:**
```yaml
apiVersion: v1
data:
  log_level: debug
kind: ConfigMap
metadata:
  creationTimestamp: "..."
  name: another-config
  namespace: default
  resourceVersion: "..."
  uid: "..."
```

## Exam Tip
For the CKAD exam, speed is critical. Creating ConfigMaps with `kubectl create configmap ... --from-literal` is much faster than writing a YAML file from scratch. Memorize this command pattern, as it's a common requirement for setting up application environments.

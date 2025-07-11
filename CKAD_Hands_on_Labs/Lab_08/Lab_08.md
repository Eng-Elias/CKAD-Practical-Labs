# CKAD Lab 8: Creating a ConfigMap from a YAML File

## Objective
Learn what a ConfigMap is and how to create one declaratively from a YAML file to store configuration data.

## What is a ConfigMap?
A ConfigMap is a Kubernetes object used to store non-confidential data in key-value pairs. The primary purpose of a ConfigMap is to decouple environment-specific configuration from your container images, which makes your applications more portable.

Instead of hard-coding configuration data (like application endpoints, feature flags, or database hosts) into your application code, you can store it in a ConfigMap. Your pods can then consume this data as environment variables, command-line arguments, or as configuration files in a volume.

## Solution
This lab demonstrates how to create a simple ConfigMap with two key-value pairs using a YAML manifest.

### Step 1: Create a YAML File for the ConfigMap
Create a file named `my-configmap.yaml`. The `data` field holds the key-value pairs.

**`my-configmap.yaml`:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  # Simple key-value pairs
  app.properties.key1: "value1"
  app.properties.key2: "value2"

  # You can also store file-like content
  config.json: | 
    {
      "apiUrl": "http://my-api-service:8080",
      "featureFlags": {
        "beta-feature-enabled": true
      }
    }
```

### Step 2: Apply the Manifest
Create the ConfigMap in the cluster using `kubectl apply`.

**Command:**
```bash
kubectl apply -f my-configmap.yaml
```

### Step 3: Verify the ConfigMap
You can list the ConfigMaps (`cm` is the shortcut) and then describe your new one to see its contents.

**Command:**
```bash
# List ConfigMaps
kubectl get cm

# Describe the ConfigMap to see the data
kubectl describe cm my-app-config
```

**Expected Output from `describe`:**
```
Name:         my-app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
app.properties.key1:
----
value1
app.properties.key2:
----
value2
config.json:
----
{
  "apiUrl": "http://my-api-service:8080",
  "featureFlags": {
    "beta-feature-enabled": true
  }
}

Events:  <none>
```

## Explanation
Creating a ConfigMap from a YAML file is the most common and recommended approach for managing application configuration declaratively. It allows you to version-control your configuration alongside your application code. Once created, this ConfigMap is ready to be consumed by pods, which is a topic covered in subsequent labs.

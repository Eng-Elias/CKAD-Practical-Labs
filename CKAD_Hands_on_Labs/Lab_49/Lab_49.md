# CKAD Lab 49: Essential YAML Data Structures for Kubernetes

## Objective
To understand the fundamental YAML data structures used in Kubernetes manifests. A solid grasp of this syntax is critical for passing the CKAD exam, as it allows you to write manifests quickly and troubleshoot common errors like `expecting a map but got a string`.

## 1. Basic Data Types
YAML includes simple data types like strings, numbers, and booleans.

```yaml
# String (quotes are optional)
key_string: "some value"

# Number
key_number: 123

# Boolean
key_boolean: true
```

## 2. Key-Value Pairs
The most basic structure in YAML is the key-value pair. Most lines in a Kubernetes manifest are key-value pairs.

```yaml
# A simple key-value pair
apiVersion: v1

# Another one
kind: Pod
```

## 3. Arrays (Lists)
An array, or list, is a collection of items where each item is denoted by a dash (`-`). This is used when a key has multiple values.

```yaml
# A list of strings for container arguments
spec:
  containers:
  - name: my-container
    image: busybox
    # This 'args' key has an array/list as its value
    args:
    - "sleep"
    - "3600"
```

## 4. Objects (Maps)
An object, which error messages often refer to as a **map**, is a collection of key-value pairs. An object is introduced when a key is followed by a colon and an indented block of new key-value pairs.

```yaml
# The 'metadata' key has an object as its value
metadata:
  # These are key-value pairs within the metadata object
  name: my-pod
  labels:
    # The 'labels' key also has an object as its value
    app: my-app
```

## 5. The Most Important Structure: Array of Objects
This is the most common and complex structure in Kubernetes. It's a list where **each item in the list is itself an object**. You can identify this pattern when a key is followed by a list of items (each starting with `-`), and each item is an indented block of key-value pairs.

The `spec.containers` field is the classic example.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  # 'containers' is an array of objects.
  containers:

  # The dash marks the beginning of the FIRST object in the array.
  - name: nginx-container
    image: nginx
    ports:
      - containerPort: 80 # 'ports' is also an array of objects

  # The dash marks the beginning of the SECOND object in the array.
  - name: busybox-container
    image: busybox
    args:
      - "sleep"
      - "1h"
```

## Exam Tip: Troubleshooting
Most YAML errors on the exam are due to incorrect indentation or mixing up these structures. If you get an error like `error converting YAML to JSON: yaml: line X: did not find expected key` or `expecting a map but got a string`, it almost always means:
-   You have an indentation error.
-   You forgot a dash (`-`) for an item in an array.
-   You added a dash where one wasn't needed.
-   You tried to define a simple value (a string) where Kubernetes expected an object (an indented block of keys).

Understanding the difference between a simple array and an **array of objects** is the key to avoiding these issues.

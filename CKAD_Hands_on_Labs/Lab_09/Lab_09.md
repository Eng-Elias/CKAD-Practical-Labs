# CKAD Lab 9: Creating a Secret from Base64 Encoded Values

## Objective
Learn what a Kubernetes Secret is and how to create one declaratively from a YAML file using base64-encoded data.

## What is a Secret?
Similar to a ConfigMap, a Secret is a Kubernetes object that stores key-value pairs. However, Secrets are intended specifically for sensitive information, such as passwords, API tokens, or TLS certificates.

Storing this data in a Secret is more secure than putting it in a ConfigMap or hard-coding it into a pod definition. Kubernetes stores Secrets using base64 encoding by default. **Important:** Base64 is an encoding scheme, not an encryption scheme. It simply prevents the data from being casually viewed in plain text. For true security, you should use additional measures like etcd encryption at rest and Role-Based Access Control (RBAC) to limit who can access the Secrets.

## Solution
This lab demonstrates how to create a Secret with a username and password defined in a YAML manifest.

### Step 1: Base64 Encode Your Data
Before creating the YAML file, you must encode your secret values into base64. You can do this easily on the command line.

**Command:**
```bash
# Encode a username 'admin'
$ echo -n 'admin' | base64
# Output: YWRtaW4=

# Encode a password 'S3cr3tP@ssw0rd'
$ echo -n 'S3cr3tP@ssw0rd' | base64
# Output: UzNjcjN0UA==c3cwcmQ=
```
**Note:** The `-n` flag in `echo` is crucial. It prevents `echo` from adding a newline character, which would otherwise be included in the base64 encoding.

### Step 2: Create a YAML File for the Secret
Create a file named `my-secret.yaml`. In the `data` field, use the base64-encoded strings you just generated.

**`my-secret.yaml`:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque # Default type, for arbitrary key-value pairs
data:
  username: YWRtaW4=
  password: UzNjcjN0UA==c3cwcmQ=
```

### Step 3: Apply the Manifest
Create the Secret in the cluster.

**Command:**
```bash
kubectl apply -f my-secret.yaml
```

### Step 4: Verify the Secret
When you `get` a secret, its data is hidden. To view the encoded data, you must get the YAML or JSON output.

**Command:**
```bash
# Get the secret's YAML definition
kubectl get secret my-app-secret -o yaml
```

To decode a value from a running secret, you can use `kubectl get` with a `jsonpath` and pipe it to the `base64` command.

**Command to decode:**
```bash
# Decode the password
kubectl get secret my-app-secret -o jsonpath='{.data.password}' | base64 --decode
# Output: S3cr3tP@ssw0rd
```

## Explanation
Creating Secrets from YAML files is the standard declarative way to manage sensitive credentials in Kubernetes. This allows you to store the encoded values in version control. Once created, the Secret can be mounted into pods as environment variables or as files in a volume, keeping the sensitive data separate from the application image.

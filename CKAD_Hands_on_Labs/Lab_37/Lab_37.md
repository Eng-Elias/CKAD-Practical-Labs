# CKAD Lab 37: Creating a Secret with Multiple Key-Value Pairs

## Objective
Create a Kubernetes `Secret` with multiple key-value pairs using a single imperative command.

## The Challenge
Create a `Secret` with the following specifications:
-   **Name:** `foo-secret`
-   **Type:** `generic`
-   **Data:**
    -   `foo` = `bar`
    -   `foo2` = `bar2`
    -   `foo3` = `bar3`

## Solution Strategy
The fastest way to create a `Secret` with literal key-value pairs is with the `kubectl create secret generic` command. The key is to use the `--from-literal` flag for each piece of data you want to include.

### Step 1: The Imperative Command
Construct a single command that specifies the secret's name and provides each key-value pair.

**Command:**
```bash
kubectl create secret generic foo-secret \
  --from-literal=foo=bar \
  --from-literal=foo2=bar2 \
  --from-literal=foo3=bar3
```

**Command Breakdown:**
-   `kubectl create secret generic foo-secret`: Specifies that we are creating a generic secret named `foo-secret`.
-   `--from-literal=key=value`: This flag is repeated for each key-value pair. The value is stored as a Base64-encoded string within the secret.

### Step 2: Verify the Secret
After creating the secret, you can verify its existence and check its data keys.

**Commands:**
```bash
# Check that the secret was created
kubectl get secret foo-secret

# Describe the secret to see its keys and their sizes
kubectl describe secret foo-secret
```

**Expected Output for `describe`:**
```
Name:         foo-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
foo:   3 bytes
foo2:  4 bytes
foo3:  4 bytes
```

Note that `kubectl describe` does not show the secret's values, only the keys and the size of the values. This is an intentional security feature.

### Step 3: (Optional) View the Secret's Data
If you need to see the actual Base64-encoded values, you can output the secret in YAML or JSON format.

**Command:**
```bash
kubectl get secret foo-secret -o yaml
```
This will show the `data` section with the Base64-encoded values, which you could then decode if necessary.

## Exam Tip
For the CKAD exam, `kubectl create secret generic --from-literal=...` is the go-to command for creating secrets quickly. Memorize this pattern, especially the ability to chain multiple `--from-literal` flags.

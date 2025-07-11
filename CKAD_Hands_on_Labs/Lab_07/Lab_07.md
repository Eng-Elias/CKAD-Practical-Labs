# CKAD Lab 7: Creating a Service Account

## Objective
Understand what a Kubernetes Service Account is and learn how to create one using a YAML manifest.

## What is a Service Account?
A Service Account provides an identity for processes that run within a Pod. When a pod needs to interact with the Kubernetes API (e.g., to list other pods, read a ConfigMap, etc.), it authenticates itself using the token associated with its Service Account.

Every namespace has a `default` Service Account that is automatically created and assigned to new pods unless another Service Account is specified. This allows pods to have a basic level of API access out of the box.

## Solution
This lab demonstrates how to create a custom Service Account. While you can create one imperatively, using a YAML file is the standard for declarative, repeatable infrastructure.

### Step 1: Create a YAML File for the Service Account
The manifest for a Service Account is one of the simplest in Kubernetes. It only requires the `apiVersion`, `kind`, and `metadata.name`.

**`my-service-account.yaml`:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
```

### Step 2: Apply the Manifest
Create the Service Account in the cluster using `kubectl apply`.

**Command:**
```bash
kubectl apply -f my-service-account.yaml
```

### Step 3: Verify the Service Account
You can list the Service Accounts in the current namespace to confirm that yours was created. `sa` is a convenient shortcut for `serviceaccount`.

**Command:**
```bash
# List all service accounts in the default namespace
kubectl get sa
```

**Expected Output:**
```
NAME          SECRETS   AGE
default       1         1h
my-app-sa     1         15s
```

### Step 4: Inspect the Service Account (Optional)
When a Service Account is created, Kubernetes automatically generates a corresponding `Secret` that holds the authentication token. You can see this relationship by describing the Service Account.

**Command:**
```bash
kubectl describe sa my-app-sa
```

**Expected Output:**
```
Name:                my-app-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   my-app-sa-token-xxxxx
Tokens:              my-app-sa-token-xxxxx
Events:              <none>
```

## Explanation
Creating a custom Service Account is the first step in implementing the principle of least privilege for your applications. Instead of using the `default` Service Account with its broad permissions, you can create dedicated accounts for each application. You would then use Role-Based Access Control (RBAC) resources (`Roles` and `RoleBindings`) to grant that specific Service Account only the permissions it needs to function.

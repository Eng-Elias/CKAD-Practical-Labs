# CKAD Lab 4: Using Namespaces to Isolate Resources

## Objective
Understand the purpose of Kubernetes namespaces and learn how to create and manage resources within different namespaces to achieve workload isolation.

## What are Namespaces?
Namespaces are a fundamental concept in Kubernetes used to create virtual clusters within a single physical cluster. They provide a scope for names, meaning a resource name must be unique within a namespace, but not across namespaces. This allows different teams, projects, or environments (e.g., `development`, `staging`, `production`) to coexist on the same cluster without interfering with each other.

## Solution
This lab demonstrates how to create two namespaces and deploy a pod with the exact same name into each one.

### Step 1: Create New Namespaces
Use the `kubectl create namespace` command to create two new namespaces, `ns1` and `ns2`.

**Command:**
```bash
# Create the first namespace
kubectl create namespace ns1

# Create the second namespace
kubectl create namespace ns2
```

**Verification:**
```bash
# List all namespaces (ns is a shortcut for namespace)
kubectl get ns
# NAME              STATUS   AGE
# default           Active   1h
# kube-node-lease   Active   1h
# kube-public       Active   1h
# kube-system       Active   1h
# ns1               Active   10s
# ns2               Active   8s
```

### Step 2: Create Identical Pods in Each Namespace
Now, create a pod named `nginx` in `ns1` and another pod with the exact same name, `nginx`, in `ns2`. Use the `-n` (or `--namespace`) flag to specify the target namespace for each command.

**Command:**
```bash
# Create the nginx pod in namespace ns1
kubectl run nginx --image=nginx -n ns1

# Create the exact same nginx pod in namespace ns2
kubectl run nginx --image=nginx -n ns2
```

### Step 3: Verify the Pods
To see resources across all namespaces, use the `-A` (or `--all-namespaces`) flag. You can then `grep` for your specific pod name to see that both instances are running in their respective namespaces.

**Command:**
```bash
# List pods in all namespaces and filter for 'nginx'
kubectl get pods -A | grep nginx
```

**Expected Output:**
```
ns1      nginx     1/1     Running   0          60s
ns2      nginx     1/1     Running   0          55s
```

## Explanation
This exercise demonstrates the core function of namespaces: resource isolation. Even though both pods are named `nginx`, they are unique objects within the cluster because they exist in different namespaces (`ns1` and `ns2`). This prevents naming conflicts and allows for the logical separation of applications and environments. When interacting with a resource in a non-default namespace, you must always use the `-n <namespace>` flag.

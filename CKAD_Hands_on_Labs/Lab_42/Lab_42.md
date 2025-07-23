# CKAD Lab 42: Creating a Pod in a Specific Namespace

## Objective
Understand how to create and manage resources within a specific Kubernetes `Namespace`. Namespaces are a fundamental mechanism for isolating groups of resources within a single cluster.

## The Challenge
1.  Create a new `Namespace` named `ns-foo`.
2.  Create a `Pod` named `pod-foo` using the `nginx:alpine` image inside the `ns-foo` namespace.

## Solution Strategy
This is a two-step process using simple imperative commands. The key is remembering to use the `--namespace` (or `-n`) flag when creating and viewing the pod.

### Step 1: Create the Namespace
First, create the namespace using the `kubectl create namespace` command.

**Command:**
```bash
kubectl create namespace ns-foo
```

**Verification:**
```bash
# List all namespaces to confirm creation
kubectl get namespaces
# or
kubectl get ns
```

### Step 2: Create the Pod in the Namespace
Next, use the `kubectl run` command to create the pod, specifying the target namespace with the `-n` flag.

**Command:**
```bash
kubectl run pod-foo --image=nginx:alpine -n ns-foo
```

**Command Breakdown:**
-   `kubectl run pod-foo`: Creates a pod named `pod-foo`.
-   `--image=nginx:alpine`: Specifies the container image to use.
-   `-n ns-foo`: **This is the crucial part.** It tells `kubectl` to create the pod in the `ns-foo` namespace instead of the `default` one.

### Step 3: Verify the Pod
If you run `kubectl get pods` without any flags, you will **not** see your new pod, because the command defaults to the `default` namespace.

**Commands to correctly view the pod:**
```bash
# Specifically request pods from the ns-foo namespace
kubectl get pods -n ns-foo

# Or, list all pods in all namespaces
kubectl get pods -A
```

**Expected Output for `kubectl get pods -n ns-foo`:**
```
NAME      READY   STATUS    RESTARTS   AGE
pod-foo   1/1     Running   0          10s
```

## Exam Tip
Forgetting the `-n` or `--namespace` flag is a very common mistake on the CKAD exam. If you create a resource and can't find it, your first thought should be: "Did I create it in the right namespace?" Always double-check the namespace requirements in the exam questions.

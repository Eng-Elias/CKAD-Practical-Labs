# CKAD Lab 21: Annotating Pods Using a Label Selector

## Objective
Learn how to add annotations to one or more running pods simultaneously by using a label selector (`-l`). This is a powerful and efficient way to add metadata to a group of related resources.

## What are Annotations?
Annotations are key-value pairs used to attach arbitrary non-identifying metadata to objects. While labels are used for selecting and filtering objects, annotations are used for a different purpose: to hold metadata that can be used by tools, libraries, or other automation.

**Key Differences:**
-   **Labels:** Used for selection and organization (e.g., `env=prod`, `app=nginx`). They have strict syntax rules.
-   **Annotations:** Used for attaching extra information (e.g., a description, a link to a monitoring dashboard, build version). They can hold larger, more complex data.

Changing an annotation does not change the pod itself or how it's scheduled. It's simply extra information for other tools or humans to use.

## Key Command: `kubectl annotate -l <selector>`
The `kubectl annotate` command adds or updates annotations. When combined with the `-l` (or `--selector`) flag, it applies the annotation to all resources that match the specified label query.

## Solution
This lab demonstrates how to create multiple pods with the same label and then annotate them all with a single command.

### Step 1: Create Pods with a Common Label
First, create two different pods but assign them the same `env=prod` label. This will allow us to target them as a group.

**Commands:**
```bash
# Create an nginx pod with two labels
kubectl run nginx --image=nginx --labels="app=web,env=prod"

# Create a hazelcast pod with the same env label
kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=db,env=prod"
```

### Step 2: Verify the Pods and Their Labels
Use `kubectl get pods --show-labels` to confirm both pods are running and have the correct labels.

**Command:**
```bash
kubectl get pods --show-labels
```

**Expected Output:**
```
NAME        READY   STATUS    RESTARTS   AGE   LABELS
hazelcast   1/1     Running   0          60s   app=db,env=prod
nginx       1/1     Running   0          90s   app=web,env=prod
```
Both pods share the `env=prod` label.

### Step 3: Annotate All Pods Matching the Label
Now, use `kubectl annotate` with a selector to add the annotation `owner=dev-team` to all pods with the label `env=prod`.

**Command:**
```bash
kubectl annotate pods -l env=prod owner='dev-team'
```

**Expected Output:**
```
pod/hazelcast annotated
pod/nginx annotated
```
Both pods were successfully annotated with a single command.

### Step 4: Verify the Annotations
Use `kubectl describe` on one of the pods to check that the annotation has been applied.

**Command:**
```bash
kubectl describe pod nginx
```

**Expected Output (snippet):**
```
Name:         nginx
Namespace:    default
...
Labels:       app=web
              env=prod
Annotations:  owner: dev-team
Status:       Running
...
```
You can clearly see the `owner: dev-team` annotation is now present.

## Exam Tip
Using selectors (`-l`) is a massive time-saver in the CKAD exam. Instead of performing an action on resources one-by-one, you can use a selector to modify them all at once. This applies not just to `annotate`, but also to `label`, `get`, `delete`, and `expose`.

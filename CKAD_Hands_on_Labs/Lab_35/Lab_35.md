# CKAD Lab 35: Creating a Deployment with Replicas

## Objective
Create a Kubernetes Deployment with a specific name, image, and number of replicas using a single imperative command.

## The Challenge
Create a Deployment with the following specifications:
-   **Name:** `foo-dep`
-   **Image:** `nginx`
-   **Replicas:** 5

## Solution Strategy
For creating a standard Deployment, the `kubectl create deployment` command is the most direct and fastest method. This is a critical command to master for the CKAD exam.

### Understanding the Resource Hierarchy
When you create a Deployment, you are actually creating three layers of resources:
1.  **Deployment:** The top-level object that manages the application's lifecycle, including updates and rollbacks.
2.  **ReplicaSet:** The Deployment automatically creates a ReplicaSet. The ReplicaSet's job is to ensure that a specified number of identical pods are running at all times.
3.  **Pods:** The ReplicaSet creates and manages the individual pods, which run the application containers.

You manage the Deployment, and the Deployment manages the rest.

### Step 1: The Imperative Command
We can create the required resources with a single line.

**Command:**
```bash
kubectl create deployment foo-dep --image=nginx --replicas=5
```

**Command Breakdown:**
-   `kubectl create deployment foo-dep`: Specifies the type of resource and its name.
-   `--image=nginx`: Sets the container image for the pods.
-   `--replicas=5`: Tells the Deployment to maintain 5 running pods.

### Step 2: Verify All Created Resources
After running the command, you can verify each layer of the hierarchy.

**Commands:**
```bash
# 1. Check the Deployment status
kubectl get deployment foo-dep

# 2. Check the ReplicaSet created by the Deployment
# Note the name will be the deployment name plus a random hash.
kubectl get replicaset

# 3. Check the Pods created by the ReplicaSet
kubectl get pods
```

**Expected Output for `get deployment`:**
```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
foo-dep   5/5     5            5           30s
```

**Expected Output for `get pods`:**
```
NAME                       READY   STATUS    RESTARTS   AGE
foo-dep-6d5b4f8c5c-5h2j4   1/1     Running   0          45s
foo-dep-6d5b4f8c5c-8b9l6   1/1     Running   0          45s
foo-dep-6d5b4f8c5c-9k4m7   1/1     Running   0          45s
foo-dep-6d5b4f8c5c-f2g6j   1/1     Running   0          45s
foo-dep-6d5b4f8c5c-z1x3c   1/1     Running   0          45s
```

## Exam Tip
`kubectl create deployment` is a non-negotiable command to know for the CKAD exam. It is significantly faster than writing a YAML file from scratch. Practice it until it becomes second nature.

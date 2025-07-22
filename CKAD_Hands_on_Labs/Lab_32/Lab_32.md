# CKAD Lab 32: Creating a Pod with a Label

## Objective
Create a pod with a specific label using a single imperative `kubectl` command.

## The Challenge
Create a pod with the following specifications:
-   **Name:** `foo-pod`
-   **Image:** `nginx`
-   **Label:** `foo=bar`

## Solution Strategy
For creating simple pods, the `kubectl run` command is the fastest method. It's important to remember this distinction, as `kubectl create` is used for many other resource types, but not for pods.

### Step 1: The `kubectl run` Command
We will use `kubectl run` and provide the pod name, image, and the desired labels using the `--labels` flag.

**Command:**
```bash
kubectl run foo-pod --image=nginx --labels=foo=bar
```

**Command Breakdown:**
-   `kubectl run foo-pod`: Specifies the name of the pod to create.
-   `--image=nginx`: Specifies the container image to use.
-   `--labels=foo=bar`: Attaches the label `foo` with the value `bar` to the pod's metadata.

### Step 2: Verify the Pod and its Label
After running the command, you can verify that the pod was created and that the label was applied correctly.

**Commands:**
```bash
# Check that the pod is running
kubectl get pods

# Check the pod's labels
kubectl get pod foo-pod --show-labels
```

**Expected Output for `--show-labels`:**
```
NAME      READY   STATUS    RESTARTS   AGE   LABELS
foo-pod   1/1     Running   0          30s   foo=bar
```

This output confirms that the pod is running and has the correct label attached.

## Exam Tip
During the CKAD exam, speed is critical. Using imperative commands like `kubectl run` to create resources like pods is much faster than writing a full YAML manifest. For any question that asks you to simply create a pod with a specific image or label, `kubectl run` should be your go-to command.

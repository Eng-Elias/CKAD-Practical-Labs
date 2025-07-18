# CKAD Lab 26 (Part 3): Practice Exam - Deployment and Verification

## Objective
Complete the practice lab by deploying the pod manifest created in the previous parts and performing a thorough verification to ensure all requirements have been met.

## Recap of the Challenge

We have created a complete `pod.yaml` manifest that defines a pod in the `foo-ns` namespace. This pod uses a `busybox` container that:
1.  Mounts an `emptyDir` volume at `/etc/foo`.
2.  Injects a value from the `foo-cm` ConfigMap as an environment variable named `MY_FOO_VAR`.
3.  Runs a continuous loop that writes the value of `$MY_FOO_VAR` into the file `/etc/foo/foo.txt`.

## Part 3 Solution: Deployment and End-to-End Verification

### Step 1: Apply the Pod Manifest
With the `pod.yaml` file from Part 2 ready, use `kubectl apply` to create the pod.

**Command:**
```bash
kubectl apply -f pod.yaml
```

**Expected Output:**
```
pod/foo-pod created
```

### Step 2: Initial Verification
Check that the pod is running successfully. Remember to specify the namespace.

**Command:**
```bash
kubectl get pod -n foo-ns
```

**Expected Output:**
```
NAME      READY   STATUS    RESTARTS   AGE
foo-pod   1/1     Running   0          30s
```

You can also use `kubectl describe pod foo-pod -n foo-ns` to review the pod's configuration, including its command, environment variables, and volume mounts, as shown in the video.

### Step 3: Final Verification (The Most Important Step)
The ultimate test is to confirm that the file is actually being written to inside the container. To do this, we will use `kubectl exec` to run a command inside the running pod.

**Command:**
```bash
# Exec into the pod and 'cat' the contents of the target file
kubectl exec -it foo-pod -n foo-ns -- cat /etc/foo/foo.txt
```

**Expected Output:**
You should see the value from the ConfigMap being printed repeatedly, confirming that the entire chain is working:
```
The value from the configmap is: bar
The value from the configmap is: bar
The value from the configmap is: bar
...
```

This confirms:
-   The pod is running.
-   The `emptyDir` volume was mounted correctly at `/etc/foo`.
-   The ConfigMap value was successfully injected as an environment variable.
-   The shell command is executing correctly and writing to the file in the mounted volume.

## Exam Tip
For the CKAD exam, verification is key. Don't just create a resource and assume it works. Use commands like `kubectl describe`, `kubectl logs`, and especially `kubectl exec` to prove that your solution meets all the requirements of the question. For a question like this, `exec`-ing into the pod to check the file's content is the only way to be 100% certain you have solved it correctly.

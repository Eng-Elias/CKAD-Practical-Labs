# CKAD Lab 31 (Part 2): Deployment Upgrade and Rollback

## Objective
Complete the deployment lifecycle lab by performing a rolling update to a new application version and then rolling back to the previous version.

## Recap
In Part 1, we created a Deployment named `foo-deploy` with 4 replicas, using the `nginx:1.16` image and a custom rolling update strategy. Now, we will put that strategy into action.

## Part 2 Solution: Upgrade and Rollback

### Step 1: Apply the Initial Deployment
First, ensure the initial deployment from Part 1 is running.

**Commands:**
```bash
# Apply the manifest from Part 1
kubectl apply -f deployment.yaml

# Verify the deployment is stable and all pods are running the correct version
kubectl describe deployment foo-deploy
```
In the description, you should see the `nginx:1.16` image and 4/4 available replicas.

### Step 2: Perform a Rolling Update (Upgrade)
The easiest way to trigger a rolling update is to change the image version using `kubectl set image`.

**Command:**
```bash
# Change the image for the 'nginx' container in the 'foo-deploy' deployment
kubectl set image deployment/foo-deploy nginx=nginx:1.17
```

### Step 3: Monitor the Rollout
Immediately after running the `set image` command, you can watch the rollout happen in real time. This is where the `maxSurge` and `maxUnavailable` parameters come into play.

**Commands:**
```bash
# Watch the status of the rollout
kubectl rollout status deployment/foo-deploy

# (In another terminal) Watch the pods being terminated and created
kubectl get pods --watch
```
You will see new pods with the `nginx:1.17` image being created and old pods with the `nginx:1.16` image being terminated, all while respecting the `maxSurge` and `maxUnavailable` constraints.

### Step 4: Check the Rollout History
Once the rollout is complete, you can view the history of changes. This is crucial for rollbacks.

**Command:**
```bash
kubectl rollout history deployment/foo-deploy
```

**Expected Output:**
```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```
Revision 2 is our current state (`nginx:1.17`), and Revision 1 is the original state (`nginx:1.16`).

### Step 5: Roll Back to the Previous Version
Now, let's undo the change and go back to the previous version. The `rollout undo` command makes this simple.

**Command:**
```bash
# Roll back to the previous revision (Revision 1)
kubectl rollout undo deployment/foo-deploy
```

This will trigger another rolling update, this time replacing the `1.17` pods with `1.16` pods.

### Step 6: Verify the Rollback
Monitor the rollback status and then check the history again.

**Commands:**
```bash
# Watch the rollback complete
kubectl rollout status deployment/foo-deploy

# Check the image version in the deployment description
kubectl describe deployment foo-deploy | grep Image
# Expected output: Image: nginx:1.16

# Check the history again
kubectl rollout history deployment/foo-deploy
```
After the rollback, you will see a new revision (e.g., Revision 3) which is a copy of the state from Revision 1.

## Exam Tip
For the CKAD exam, you must be fast with these commands:
-   `kubectl set image deployment/...`
-   `kubectl rollout status deployment/...`
-   `kubectl rollout history deployment/...`
-   `kubectl rollout undo deployment/...`

Practice them until they are muscle memory. They are almost guaranteed to appear on the exam.

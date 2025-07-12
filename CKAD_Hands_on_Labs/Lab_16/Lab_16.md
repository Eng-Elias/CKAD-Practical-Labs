# CKAD Lab 16: Monitoring Deployment Rollout Status

## Objective
Learn how to monitor the real-time status of a Kubernetes deployment as it's being rolled out. This is essential for ensuring that updates are proceeding as expected and for debugging issues during a deployment.

## Key Command: `kubectl rollout status`

The `kubectl rollout status` command allows you to watch the status of a deployment until it is complete. It provides live updates on the number of replicas that are updated, available, and ready.

This is particularly useful for large deployments or in environments where new pods might take some time to start up. Instead of repeatedly running `kubectl get pods`, you can use `rollout status` for a clear, concise summary of the deployment's progress.

## Solution
To make the rollout process observable, this lab creates a deployment with a high number of replicas (100). This slows down the process enough to see the status updates in real time.

### Step 1: Create a Deployment with Many Replicas
First, we create a deployment named `my-dp` using the `nginx` image with 100 replicas.

**Command:**
```bash
# Create a deployment with 100 nginx pods
kubectl create deployment my-dp --image=nginx --replicas=100
```

### Step 2: Immediately Monitor the Rollout Status
As soon as the deployment is created, use the `kubectl rollout status` command to watch its progress. The command will exit once the deployment is successfully completed.

**Command:**
```bash
# Watch the rollout status of the 'my-dp' deployment
kubectl rollout status deployment/my-dp
```

**Expected Output:**
```
Waiting for deployment "my-dp" rollout to finish: 0 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 48 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 95 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 96 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 97 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 98 of 100 updated replicas are available...
Waiting for deployment "my-dp" rollout to finish: 99 of 100 updated replicas are available...
deployment "my-dp" successfully rolled out
```

### Step 3: Verify the Pods (Optional)
Once the rollout is complete, you can verify that all 100 pods are running.

**Command:**
```bash
kubectl get pods
```

### Step 4: Clean Up
Finally, delete the deployment to remove all the pods and associated resources.

**Command:**
```bash
kubectl delete deployment my-dp
```

## Exam Tip
In the CKAD exam, you might be asked to perform an update and verify that it was successful. Using `kubectl rollout status` is the most direct and efficient way to confirm that a deployment has completed successfully without having to manually count pods.

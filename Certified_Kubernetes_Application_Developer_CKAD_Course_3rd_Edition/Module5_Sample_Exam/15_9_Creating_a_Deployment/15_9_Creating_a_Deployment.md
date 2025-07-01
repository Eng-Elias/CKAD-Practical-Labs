# 15.9 Creating a Deployment

This section provides a solution for the sample exam assignment on creating a deployment with specific replica, labeling, and update strategy requirements.

## Task

1.  Write a manifest file named `nginx-exam.yaml`.
2.  The deployment should start with **5 replicas** running the `nginx:1.18` image.
3.  Each Pod created by the deployment must have the label `type: webshop`.
4.  The Deployment object itself should have the label `service: nginx`.
5.  Configure the update strategy such that during an update, a maximum of **8 Pods** can be running, and a minimum of **3 Pods** must always be available.
6.  After creating the deployment, update the image to the latest version of NGINX.

---

## Solution Walkthrough

This task combines creating a deployment from scratch with configuring a custom rolling update strategy.

### 1. Generate the Initial YAML

Using `kubectl create deploy` with `--dry-run` is the fastest way to generate a valid starting manifest.

```bash
kubectl create deploy nginx-exam --image=nginx:1.18 --dry-run=client -o yaml > nginx-exam.yaml
```

### 2. Modify the YAML File

Now, edit `nginx-exam.yaml` to meet all the requirements.

#### A. Set Replicas and Labels

*   Change `spec.replicas` to `5`.
*   Add the `service: nginx` label to the Deployment's `metadata.labels`.
*   Add the `type: webshop` label to the Pod template's `spec.template.metadata.labels`.

#### B. Configure the Update Strategy

This is the most complex part. You need to add a `strategy` block under `spec`. Use `kubectl explain deployment.spec.strategy --recursive` to find the correct fields: `rollingUpdate`, `maxSurge`, and `maxUnavailable`.

*   **`maxSurge`**: The number of pods that can be created *above* the desired count. The goal is a maximum of 8 pods. With a desired state of 5, this means we can surge by 3 (5 + 3 = 8). So, `maxSurge: 3`.
*   **`maxUnavailable`**: The number of pods that can be unavailable during the update. The requirement is that 3 pods must always be available. With a desired state of 5, this means a maximum of 2 can be unavailable (5 - 2 = 3). So, `maxUnavailable: 2`.

Here is the complete, correct YAML:

```yaml
# nginx-exam.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-exam
  labels:
    service: nginx # Label for the Deployment itself
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-exam
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx-exam
        type: webshop # Label for the Pods
    spec:
      containers:
      - image: nginx:1.18
        name: nginx
```

### 3. Create and Verify the Deployment

Apply the manifest file.

```bash
kubectl create -f nginx-exam.yaml
```

Check the status. The transcription notes a common mistake: if you forget to set the `replicas` in the file, only one pod will start. You would then need to delete the deployment, fix the file, and create it again.

```bash
# Verify the pods are created
kubectl get pods --selector app=nginx-exam
```

You should see 5 pods being created.

### 4. Update the Deployment Image

Use `kubectl set image` to trigger the rolling update.

```bash
kubectl set image deployment/nginx-exam nginx=nginx:latest
```

### 5. Verify the Rolling Update

While the update is in progress, run `kubectl get pods --selector app=nginx-exam` again. You will see a mix of old and new pods. The total number of pods will not exceed 8, and the number of available pods will not drop below 3, demonstrating that the `maxSurge` and `maxUnavailable` settings are working as intended.

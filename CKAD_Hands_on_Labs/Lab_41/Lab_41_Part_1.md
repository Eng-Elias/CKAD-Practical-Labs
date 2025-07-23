# CKAD Lab 41 (Part 1): Creating and Labeling a Deployment

## Objective
This is the first part of a three-part lab covering the full application networking stack in Kubernetes. In this part, we will create the backend workload using a Deployment and apply the necessary labels for service discovery.

## The Full Challenge (Multi-Part)

1.  **Part 1: Create a Deployment** named `deploy-foo` using the `redis:alpine` image, with 3 replicas and the label `app=foo`.
2.  **Part 2: Expose the Deployment** internally with a `ClusterIP` Service named `svc-foo` on port `6379`.
3.  **Part 3: Create a Network Policy** that allows ingress traffic only from specific pods.

## Part 1 Solution: The Deployment
The fastest way to create a deployment is with an imperative command. However, `kubectl create deployment` does not have a `--labels` flag for custom labels, so we must add the label in a second step.

### Step 1: Create the Deployment
First, create the deployment with the specified name, image, and replica count. We also expose the container port, which is good practice.

**Command:**
```bash
kubectl create deployment deploy-foo --image=redis:alpine --replicas=3 --port=6379
```

### Step 2: Add the Required Label
With the deployment created, use the `kubectl label` command to add the custom label. This label will be used by the Service in the next part to identify which pods to send traffic to.

**Command:**
```bash
kubectl label deployment deploy-foo app=foo
```

**Command Breakdown:**
-   `kubectl label`: The command to add, update, or remove labels.
-   `deployment deploy-foo`: The resource type and name to target.
-   `app=foo`: The key-value pair for the label.

### Step 3: Verify the Deployment and Label
Check that the deployment is running and that the label has been successfully applied.

**Commands:**
```bash
# Check the deployment status
kubectl get deployment deploy-foo

# Describe the deployment to see its labels
kubectl describe deployment deploy-foo
```

In the `describe` output, under the `Labels` section, you should see `app=foo`.

## Exam Tip
Remember this two-step process for creating labeled deployments imperatively. It's much faster than writing a full YAML manifest. Forgetting to add the label is a common mistake that will cause your Service (in Part 2) to fail because its selector won't find any pods.

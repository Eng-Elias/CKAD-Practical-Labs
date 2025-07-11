# CKAD Lab 5: How Deployment Pods are Distributed Across Nodes

## Objective
Observe how the Kubernetes scheduler automatically distributes the pods of a Deployment evenly across the available nodes in a cluster to ensure high availability and resilience.

## The Concept: Automatic Pod Scheduling
When you create a `Deployment` with multiple replicas, the Kubernetes scheduler is responsible for deciding which node each pod should run on. By default, the scheduler's goal is to spread the pods across the nodes. This prevents a single node failure from taking down all replicas of your application, which is a core principle of high availability.

## Solution
This lab demonstrates the scheduler's behavior by creating a Deployment with a number of replicas equal to the number of nodes in the cluster.

### Step 1: Check the Nodes
First, see how many nodes are in your cluster.

**Command:**
```bash
kubectl get nodes
```
**Example Output (for a 5-node cluster):**
```
NAME           STATUS   ROLES    AGE   VERSION
gke-worker-1   Ready    <none>   2h    v1.25.0
gke-worker-2   Ready    <none>   2h    v1.25.0
gke-worker-3   Ready    <none>   2h    v1.25.0
gke-worker-4   Ready    <none>   2h    v1.25.0
gke-worker-5   Ready    <none>   2h    v1.25.0
```

### Step 2: Create a Deployment with Multiple Replicas
Create a new Deployment with a number of replicas matching the number of nodes (e.g., 5).

**Command:**
```bash
# Create a deployment named 'my-app' with 5 replicas
kubectl create deployment my-app --image=nginx --replicas=5
```

### Step 3: Verify Pod Distribution
Use the `kubectl get pods -o wide` command to see the list of pods and which node each one has been scheduled on. You will see that the scheduler has placed one pod on each available node.

**Command:**
```bash
kubectl get pods -o wide
```

**Expected Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE   IP          NODE           NOMINATED NODE   READINESS GATES
my-app-6c8f4d4b7b-abcde   1/1     Running   0          30s   10.244.1.5   gke-worker-2   <none>           <none>
my-app-6c8f4d4b7b-fghij   1/1     Running   0          30s   10.244.2.6   gke-worker-3   <none>           <none>
my-app-6c8f4d4b7b-klmno   1/1     Running   0          30s   10.244.0.4   gke-worker-1   <none>           <none>
my-app-6c8f4d4b7b-pqrst   1/1     Running   0          30s   10.244.4.8   gke-worker-5   <none>           <none>
my-app-6c8f4d4b7b-uvwxyz   1/1     Running   0          30s   10.244.3.7   gke-worker-4   <none>           <none>
```

## Explanation
This automatic and even distribution of pods is a key feature of Kubernetes. The scheduler makes intelligent decisions based on available resources and default scheduling policies to maximize application uptime. You don't need to manually configure this behavior; it's part of the built-in reliability of the Kubernetes infrastructure. For more advanced control over where pods are placed, you can use features like `nodeAffinity`, `podAffinity`, `taints`, and `tolerations`.

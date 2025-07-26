# CKAD Lab 48 (Part 2): Labeling and Exposing a Deployment as a NodePort Service

## Objective
With the deployment running, the final steps are to label it for service discovery and expose it externally using a `NodePort` service. This makes the application accessible on a specific port on every node in the cluster.

## The Challenge
1.  Add the label `app=my-app` to the `foo-dp` deployment.
2.  Expose the deployment as a `NodePort` service with the following properties:
    -   Name: `foo-nodeport-service`
    -   Service Port: `80`
    -   Pod Target Port: `80` (since it's an `nginx` container)
    -   NodePort: `30001`

## Part 2 Solution: Labeling and Exposing

### Step 1: Label the Deployment
The service will use this label as a `selector` to find the pods it should send traffic to.

**Command:**
```bash
kubectl label deployment foo-dp app=my-app
```

**Verification:**
```bash
kubectl describe deployment foo-dp | grep -i labels
# Labels: app=my-app
```

### Step 2: Expose the Deployment as a NodePort Service
A `NodePort` service opens a specific port on all nodes in the cluster, and any traffic sent to this port is forwarded to the service.

While `kubectl expose` is fast, it **cannot** be used to assign a *specific* `nodePort` (like `30001`). The imperative command will assign a random high-numbered port. To meet the specific requirement of using port `30001`, we must generate the YAML and edit it.

**Workflow for a Specific NodePort:**

**1. Generate the Service YAML:**
Use `kubectl expose` with `--dry-run` to create a template.

```bash
kubectl expose deployment foo-dp --type=NodePort --port=80 --target-port=80 --name=foo-nodeport-service --dry-run=client -o yaml > service.yaml
```

**2. Edit the YAML to Add the `nodePort`:**
Open `service.yaml` and add the `nodePort: 30001` line within the `ports` section.

**`service.yaml` (Final):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: foo-nodeport-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    # Manually add this line to specify the exact NodePort
    nodePort: 30001
  selector:
    app: my-app
  type: NodePort
```

**3. Apply the Final Manifest:**
```bash
kubectl apply -f service.yaml
```

### Step 3: Verify the Service
Check that the service has been created with the correct ports.

**Command:**
```bash
kubectl get service foo-nodeport-service
```

**Expected Output:**
```
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
foo-nodeport-service   NodePort   10.108.144.20   <none>        80:30001/TCP   30s
```
The `PORT(S)` column shows that the service's internal port `80` is mapped to the external `NodePort` `30001`.

## Exam Tip
Remember the limitation of `kubectl expose`: it's great for quickly creating `ClusterIP` or `NodePort` services with random ports, but if an exam question requires a **specific `nodePort`**, you must use the generate-and-edit YAML workflow. This is a common way to test if you understand the underlying service spec.

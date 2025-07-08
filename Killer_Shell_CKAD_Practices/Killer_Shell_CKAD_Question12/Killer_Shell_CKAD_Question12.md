# CKAD Practice Question 12: Persistent Volumes and Deployments

## Question
You need to set up persistent storage for an application in the `earth` namespace.

**Tasks**:
1. **Create a PersistentVolume (PV)**:
   - Name: `pv-earth-data`
   - Capacity: 2Gi
   - Access Mode: ReadWriteOnce
   - Host Path: `/volumes/data`
   - No storage class should be defined

2. **Create a PersistentVolumeClaim (PVC)**:
   - Name: `pvc-earth-data`
   - Namespace: `earth`
   - Request: 2Gi storage
   - Access Mode: ReadWriteOnce
   - No storage class should be defined
   - Ensure it binds to the PV correctly

3. **Create a Deployment**:
   - Name: `project-earth-flower`
   - Namespace: `earth`
   - Image: `httpd:2.4.41`
   - Mount the PVC at `/tmp/project-data`

**Important Notes**:
- All resources should be created in the `earth` namespace unless specified otherwise
- Verify that the PVC binds to the PV successfully
- Ensure the deployment has the correct volume mounts

## Solution

### Step 1: Create the PersistentVolume (PV)
Create a file named `pv-earth.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-earth-data
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /volumes/data
  persistentVolumeReclaimPolicy: Retain
```

Apply the PV:
```bash
kubectl apply -f pv-earth.yaml
```

### Step 2: Create the PersistentVolumeClaim (PVC)
Create a file named `pvc-earth.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-earth-data
  namespace: earth
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

Apply the PVC:
```bash
kubectl apply -f pvc-earth.yaml -n earth
```

### Step 3: Verify PV and PVC Binding
```bash
# Check PV status
kubectl get pv pv-earth-data

# Check PVC status
kubectl get pvc pvc-earth-data -n earth
```

### Step 4: Create the Deployment
Create a file named `deployment-earth.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: project-earth-flower
  namespace: earth
  labels:
    app: earth-flower
spec:
  replicas: 1
  selector:
    matchLabels:
      app: earth-flower
  template:
    metadata:
      labels:
        app: earth-flower
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.41
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data-storage
          mountPath: /tmp/project-data
      volumes:
      - name: data-storage
        persistentVolumeClaim:
          claimName: pvc-earth-data
```

Apply the deployment:
```bash
kubectl apply -f deployment-earth.yaml -n earth
```

### Step 5: Verify the Deployment
```bash
# Check deployment status
kubectl get deployment project-earth-flower -n earth

# Check pod status
kubectl get pods -n earth -l app=earth-flower

# Verify volume mounting
kubectl describe pod -n earth -l app=earth-flower
```

## Explanation

### Key Concepts
1. **PersistentVolume (PV)**: Cluster resource for storage that has been provisioned by an administrator
2. **PersistentVolumeClaim (PVC)**: A request for storage by a user, which gets bound to a PV
3. **Access Modes**: Define how the volume can be mounted (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
4. **Storage Classes**: Allow for dynamic provisioning of storage (not used in this case)
5. **Volume Mounts**: How containers access the storage in pods

### Why This Solution Works
- The PV is created with the exact requested capacity and access mode
- The PVC requests matching storage requirements, allowing it to bind to the PV
- The deployment correctly references the PVC in its volume configuration
- The container has the volume mounted at the specified path

### Exam Tips
1. **PV and PVC Matching**: Ensure the PVC's storage request and access mode match an available PV
2. **Namespaces**: Remember to specify the namespace for namespaced resources
3. **Verification**: Always verify that PVCs are bound and pods are running with the correct volumes
4. **YAML Structure**: Pay attention to indentation in YAML files, especially for volume mounts
5. **Cleanup**: Remember to delete resources in the correct order (Pods → PVCs → PVs)

### Common Mistakes to Avoid
- Mismatched access modes between PV and PVC
- Forgetting to specify the namespace for namespaced resources
- Incorrect volume mount paths in the container
- Not verifying that the PVC is bound before creating the deployment
- Using wrong API versions for the resources

## Additional Practice
1. Create a StorageClass and modify the solution to use dynamic provisioning
2. Configure the PV to use a different type of storage (e.g., NFS, AWS EBS)
3. Set up a StatefulSet instead of a Deployment for stateful applications
4. Configure resource limits and requests for the deployment
5. Add readiness and liveness probes to the container

## Related Commands
```bash
# Get detailed information about a PV
kubectl describe pv pv-earth-data

# Get detailed information about a PVC
kubectl describe pvc pvc-earth-data -n earth

# Check events in the namespace
kubectl get events -n earth

# Get logs from the pod
kubectl logs -n earth -l app=earth-flower

# Delete resources (in the correct order)
kubectl delete deployment project-earth-flower -n earth
kubectl delete pvc pvc-earth-data -n earth
kubectl delete pv pv-earth-data
```
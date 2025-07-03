# 15.12 Using Storage

## Assignment
Create the following resources in the `ckad-1312` namespace:
1. A PersistentVolume named `1312-pv` with 2Gi storage and `ReadWriteMany` access mode
2. A PersistentVolumeClaim named `1312-pvc` requesting 1Gi storage with `ReadWriteMany` access mode
3. A Pod named `1312-pod` that uses the PVC, runs nginx, and mounts the volume at `/webdata`

## Solution Walkthrough

### 1. Create the Namespace
```bash
kubectl create namespace ckad-1312
```

### 2. Create the Storage Resources
Create a file named `storage-config.yaml` with the following content:

```yaml
---
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: 1312-pv
  namespace: ckad-1312
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/data"
---
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: 1312-pvc
  namespace: ckad-1312
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: 1312-pod
  namespace: ckad-1312
spec:
  volumes:
    - name: web-data
      persistentVolumeClaim:
        claimName: 1312-pvc
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/web-data"
          name: web-data
```

Apply the configuration:
```bash
kubectl apply -f storage-config.yaml
```

### 3. Verify the Resources

Check the PersistentVolume:
```bash
kubectl get pv 1312-pv -n ckad-1312
```

Check the PersistentVolumeClaim:
```bash
kubectl get pvc 1312-pvc -n ckad-1312
```

Check the Pod status:
```bash
kubectl get pod 1312-pod -n ckad-1312
```

### 4. Verify Volume Mount

Check the volume mount in the pod:
```bash
kubectl describe pod 1312-pod -n ckad-1312 | grep -A 5 Mounts
```

## Common Issues and Solutions

1. **PV Not Bound**
   - Ensure the storage class, access modes, and storage size in PV and PVC match
   - Check if the PV's `storageClassName` is set to `manual` to match the PVC

2. **Pod Not Starting**
   - Verify the PVC is in `Bound` status before creating the pod
   - Check pod events with `kubectl describe pod 1312-pod -n ckad-1312`

3. **Host Path Permission Issues**
   - On Minikube, you might need to create the host path directory:
     ```bash
     minikube ssh "sudo mkdir -p /mnt/data && sudo chmod 777 /mnt/data"
     ```

## Exam Tips

1. **Key Concepts**
   - PVs are cluster resources, PVCs are namespaced
   - Access modes must match between PV and PVC
   - Storage class must match for dynamic provisioning

2. **Verification**
   - Always verify PV and PVC status are `Bound`
   - Check pod events if it's not starting
   - Use `kubectl describe` for detailed information

3. **Cleanup**
   Delete resources in the correct order:
   ```bash
   kubectl delete pod 1312-pod -n ckad-1312
   kubectl delete pvc 1312-pvc -n ckad-1312
   kubectl delete pv 1312-pv
   kubectl delete namespace ckad-1312
   ```

## Additional Verification

1. **Write Test File**
   ```bash
   kubectl exec -it 1312-pod -n ckad-1312 -- /bin/sh -c "echo 'test' > /webdata/test.txt"
   ```

2. **Verify File on Host (Minikube)**
   ```bash
   minikube ssh "cat /mnt/data/test.txt"
   ```

## Conclusion

This exercise demonstrates how to set up persistent storage in Kubernetes using PVs and PVCs. The key is ensuring the PV and PVC specifications match, particularly the storage class, access modes, and storage size.
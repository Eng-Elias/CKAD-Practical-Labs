# 6.4 Understanding and Configuring SecurityContext

## Introduction to SecurityContext

SecurityContext in Kubernetes defines privilege and access control settings for pods and containers. It's a crucial aspect of Kubernetes security that helps enforce security policies at different levels.

### Key Components
- **Pod-level SecurityContext**: Applies to all containers in the pod
- **Container-level SecurityContext**: Specific to individual containers
- **Security Policies**: Define what a pod or container can do

## SecurityContext Settings

### User and Group IDs
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: ["sh", "-c", "sleep 1h"]
    securityContext:
      runAsUser: 2000
      runAsGroup: 4000
```

### Privilege Escalation
```yaml
securityContext:
  allowPrivilegeEscalation: false
  privileged: false
  capabilities:
    add: ["NET_ADMIN", "SYS_TIME"]
    drop: ["ALL"]
```

### Read-Only Root Filesystem
```yaml
securityContext:
  readOnlyRootFilesystem: true
  runAsNonRoot: true
```

## Practical Examples

### Example 1: Non-Root Container
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - name: main
    image: nginx:1.19
    ports:
    - containerPort: 80
```

### Example 2: Enhanced Security
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: secure-container
    image: nginx:1.19
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

## Troubleshooting SecurityContext

### Common Issues

#### 1. Permission Denied Errors
```bash
# Check pod events
kubectl describe pod <pod-name>

# Look for errors like:
# "container has runAsNonRoot and image has non-numeric user"
# "container has runAsNonRoot and image will run as root"
```

#### 2. Container Failing to Start
```bash
# Check container logs
kubectl logs <pod-name> --previous

# Common issues:
# - Missing volume permissions
# - Insufficient capabilities
# - Read-only filesystem violations
```

### Debugging Tips

#### Check Current Security Context
```bash
# Get pod security context
kubectl get pod <pod-name> -o jsonpath='{.spec.securityContext}'

# Get container security context
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].securityContext}'
```

#### Test with Ephemeral Containers
```bash
# Add debug container to running pod
kubectl debug -it <pod-name> --image=busybox:1.28 --target=<container-name>

# Inside container, check:
# id
# ls -la /
# mount
```

## Best Practices

1. **Run as Non-Root**
   - Always set `runAsNonRoot: true`
   - Specify non-root user with `runAsUser`

2. **Least Privilege**
   - Drop all capabilities and add only required ones
   - Set `allowPrivilegeEscalation: false`

3. **Filesystem Security**
   - Use `readOnlyRootFilesystem: true` when possible
   - Mount volumes with appropriate permissions

4. **Resource Constraints**
   - Set resource requests and limits
   - Use `fsGroup` for shared volumes

5. **Network Security**
   - Apply network policies
   - Restrict host namespaces

## Advanced SecurityContext Features

### Seccomp Profiles
```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

### AppArmor Annotations
```yaml
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/<container-name>: localhost/<profile-name>
```

### SELinux Options
```yaml
securityContext:
  seLinuxOptions:
    level: "s0:c123,c456"
```

## Real-world Example: Secure Database Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-db
  labels:
    app: database
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: postgres
    image: postgres:13
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
    env:
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: db-data
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: db-data
    persistentVolumeClaim:
      claimName: db-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## Next Steps
- Learn about Pod Security Policies (PSP) or Pod Security Standards (PSS)
- Explore Network Policies for pod-to-pod communication
- Study RBAC for fine-grained access control
- Understand service meshes for advanced security features

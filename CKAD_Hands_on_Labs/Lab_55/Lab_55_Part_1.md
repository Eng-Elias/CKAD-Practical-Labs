# CKAD Lab 55 (Part 1): Initial Setup for a Pod with Secret Volume and Node Affinity

## Objective
This is a complex, multi-part lab that combines several core Kubernetes concepts into a single challenge. The goal is to create a pod that mounts a secret as a volume and is scheduled to a specific node using `nodeAffinity`.

## The Challenge
1.  Create a `Secret` named `foo-secret`.
2.  Create a `Namespace` named `foo-ns`.
3.  Create a `Pod` in the `foo-ns` namespace with the following properties:
    -   Uses the `busybox` image.
    -   Runs the command `sleep 3600`.
    -   Mounts the `foo-secret` at the path `/etc/foo`.
    -   Is scheduled only on `node01`.

## Part 1 Solution: Creating Prerequisite Resources
Before we can create the pod, we need to set up the namespace and the secret it will consume. We also generate a template for the pod manifest.

### Step 1: Create the Namespace
All resources for this lab will live in a dedicated namespace.

**Command:**
```bash
kubectl create namespace foo-ns
```

### Step 2: Create the Secret
We can create a simple, generic secret imperatively. The content of the secret doesn't matter for this lab, only that it exists.

**Command:**
```bash
kubectl create secret generic foo-secret --from-literal=username=admin -n foo-ns
```

**Verification:**
```bash
kubectl get secret foo-secret -n foo-ns
```

### Step 3: Generate the Pod Manifest Template
To build the complex pod spec, we start by generating a basic manifest using `kubectl run` with dry-run flags.

**Command:**
```bash
kubectl run foo-pod --image=busybox -n foo-ns --dry-run=client -o yaml > pod.yaml
```

This command gives us a valid `pod.yaml` file that we can now edit to add the more complex fields.

**`pod.yaml` (Initial Template):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: foo-pod
  name: foo-pod
  namespace: foo-ns
spec:
  containers:
  - image: busybox
    name: foo-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Next Steps
With the initial setup complete, Part 2 will focus on modifying this YAML file to add:
1.  The `sleep` command.
2.  The `volume` and `volumeMount` definitions to mount the secret.

# CKAD Lab 55 (Part 3): Adding Node Affinity and Final Verification

## Objective
Complete the pod manifest by adding a `nodeAffinity` rule to ensure the pod is scheduled exclusively on `node01`. This part reinforces the use of `kubectl explain` and iterative error correction to build complex YAML structures.

## Part 3 Solution: Adding the `nodeAffinity` Rule

### Step 1: Discover the `nodeAffinity` Structure
As we've seen in previous labs, `kubectl explain` is the most reliable way to find the correct YAML path for complex fields.

**The Path:**
`pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions`

This path leads to the fields we need: `key`, `operator`, and `values`.

### Step 2: Add the Affinity Rule to the Manifest
We add the `affinity` block to the `pod.spec`. As demonstrated in the video, a fast way to build this is to assume all fields are maps and then fix the errors reported by `kubectl apply`.

The API server will report that `nodeSelectorTerms` and `matchExpressions` expect arrays, which guides us to add the dashes (`-`) in the correct places.

### The Final, Complete `pod.yaml`
This manifest combines all the requirements from the lab: the correct namespace, image, command, secret volume mount, and node affinity.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo-pod
  namespace: foo-ns
spec:
  containers:
  - name: foo-container
    image: busybox
    command: ["sleep"]
    args: ["3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: foo-secret
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

### Step 3: Apply and Verify
Now we apply the final manifest and verify that the pod is created and scheduled on the correct node.

**Commands:**
```bash
# Apply the final manifest
kubectl apply -f pod.yaml

# Wait a few moments for the pod to be scheduled and running

# Verify the pod is running and check which node it's on
kubectl get pod foo-pod -n foo-ns -o wide
```

**Expected Output:**
The output of the `get pod` command should show the `foo-pod` with a `STATUS` of `Running`, and the `NODE` column should show `node01`.

## Lab Summary
This comprehensive lab is a perfect example of a difficult CKAD exam question. It required you to:
1.  Create prerequisite resources (`Namespace`, `Secret`).
2.  Generate a base manifest using `kubectl run --dry-run`.
3.  Modify the manifest to add a `command`.
4.  Define and mount a `Secret` as a `volume`.
5.  Label a node.
6.  Use `kubectl explain` to build a `nodeAffinity` rule.
7.  Interpret and fix API server validation errors.

Mastering each of these individual skills is the key to successfully tackling complex, multi-part challenges.

# CKAD Lab 41 (Part 3): Securing a Deployment with a Network Policy

## Objective
In this final part, we will secure our `deploy-foo` deployment by creating a `NetworkPolicy` that enforces a default-deny stance and only allows ingress traffic from specific pods.

## The Challenge
-   Create a `NetworkPolicy` named `netpol-foo`.
-   This policy should apply to the pods in our `deploy-foo` deployment.
-   It should only allow **ingress** traffic from pods that have the label `app=client`.
-   The traffic should be allowed on TCP port `6379`.

## Part 3 Solution: The Network Policy
`NetworkPolicies` are the Kubernetes firewall. By default, all pods in a namespace can talk to each other. Once you apply a `NetworkPolicy` to a pod, it becomes isolated, and only the traffic explicitly allowed by the policy will be accepted.

### Step 1: The Network Policy Manifest
The best practice is to start with a template from the official Kubernetes documentation and modify it to fit the requirements.

**`netpol.yaml`:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-foo
spec:
  # 1. TARGET: Apply this policy to pods with the label 'app=foo'
  podSelector:
    matchLabels:
      app: foo

  # 2. POLICY TYPE: This policy will only affect Ingress (incoming) traffic.
  policyTypes:
    - Ingress

  # 3. RULES: Define the allowed ingress traffic.
  ingress:
    - from:
        # 4. SOURCE: Allow traffic ONLY from pods with the label 'app=client'
        - podSelector:
            matchLabels:
              app: client
      ports:
        # 5. PORT: Allow traffic ONLY to TCP port 6379 on the target pods.
        - protocol: TCP
          port: 6379
```

**How it Works:**
1.  `spec.podSelector`: This selects the pods that the policy will protect. In our case, it's the pods from `deploy-foo`, which we labeled with `app=foo` in Part 1.
2.  `spec.policyTypes`: We specify `Ingress` because we only care about controlling incoming traffic.
3.  `spec.ingress.from.podSelector`: This is the core of the rule. It defines the **source** of allowed traffic. Any pod with the label `app=client` will be allowed to connect.
4.  `spec.ingress.ports`: This further refines the rule, ensuring that allowed pods can only connect to port `6379`.

### Step 2: Apply and Verify the Policy
Apply the manifest and describe the policy to ensure it was created correctly.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f netpol.yaml

# Describe the policy to check its rules
kubectl describe networkpolicy netpol-foo
```

### Step 3: How to Test (Conceptual)
To test this, you would need a client pod:
1.  **Without the label:** `kubectl run test-client --image=busybox -- sleep 3600`. An attempt to connect from this pod to `svc-foo` on port `6379` would time out.
2.  **With the label:** `kubectl run test-client-allowed --image=busybox --labels="app=client" -- sleep 3600`. A connection attempt from this pod would succeed.

## Exam Tip
`NetworkPolicy` questions are common. Always go to the official Kubernetes documentation during the exam, find the sample `NetworkPolicy` manifest, and edit it. Do not try to write one from memory. Understand the difference between `podSelector` (the target) and `ingress.from.podSelector` (the source).

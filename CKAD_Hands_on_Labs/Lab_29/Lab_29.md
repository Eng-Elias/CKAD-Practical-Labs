# CKAD Lab 29: Creating an Ingress with Multiple Rules

## Objective
Create a single Kubernetes Ingress resource that defines multiple routing rules to direct traffic to different backend services based on the requested hostname and path.

## The Challenge
Assume the following resources already exist in your cluster:
-   A Service named `service1` exposing port `11111`.
-   A Service named `service2` exposing port `22222`.

Your task is to create a single Ingress resource named `multi-rule-ingress` that implements the following two rules:

-   **Rule 1:** Traffic for `fqdn1.com/endpoint1` should be routed to `service1` on port `11111`.
-   **Rule 2:** Traffic for `fqdn2.com/endpoint2` should be routed to `service2` on port `22222`.

## Solution Strategy

The key to this lab is understanding that the `spec.rules` field in an Ingress manifest is an array. Each item in the array is a complete rule block, starting with a hyphen (`-`). We will create a manifest with two items in the `rules` array.

### Step 1: Start with a Single-Rule Ingress Manifest
As in the previous lab, start with a basic Ingress example from the Kubernetes documentation.

### Step 2: Build the Multi-Rule Manifest
Create a file named `multi-rule-ingress.yaml` and add the two rules. You can create the first rule and then simply copy, paste, and edit it to create the second one.

**`multi-rule-ingress.yaml`:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-rule-ingress
spec:
  rules:
  # --- Rule 1 ---
  - host: "fqdn1.com"
    http:
      paths:
      - path: /endpoint1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 11111

  # --- Rule 2 ---
  - host: "fqdn2.com"
    http:
      paths:
      - path: /endpoint2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 22222
```

**Key Points:**
-   The `rules:` field contains a list (array) of rule objects.
-   Each rule starts with a `- host: ...` block.
-   Each rule is independent and directs traffic for its specified host and path to its configured backend service and port.

### Step 3: Apply and Verify
Apply the manifest and use `kubectl describe` to confirm that both rules have been correctly configured.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f multi-rule-ingress.yaml

# Describe the Ingress to see the rules
kubectl describe ingress multi-rule-ingress
```

**Verification Checklist:**
When you describe the Ingress, you should see both rules listed clearly:

-   **Rule 1:** Host `fqdn1.com`, Path `/endpoint1`, Backend `service1:11111`
-   **Rule 2:** Host `fqdn2.com`, Path `/endpoint2`, Backend `service2:22222`

## Exam Tip
Being able to quickly create multi-rule Ingress resources is a valuable skill. The fastest way to do this in the exam is to write the YAML for the first rule, then copy and paste that block and edit the `host`, `path`, `service.name`, and `service.port.number` for the subsequent rules. This minimizes typos and saves time.

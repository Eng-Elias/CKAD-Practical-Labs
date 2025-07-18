# CKAD Lab 27: Creating a Basic Ingress Resource

## Objective
Create a Kubernetes Ingress resource to route external traffic from a specific host and path to an existing backend service.

## The Challenge
Assume the following resources already exist in your cluster:
-   A Service named `my-cs`.
-   This service exposes a port `5678`.

Your task is to create an Ingress resource that directs traffic under these conditions:
-   **Host:** `foobar.com`
-   **Path:** `/foo-endpoint`
-   **Backend:** The `my-cs` service on port `5678`.

## Solution Strategy
For complex resources like Ingress, the fastest approach during the CKAD exam is to find a suitable example in the official Kubernetes documentation and adapt it to the specific requirements.

### Step 1: Find an Ingress Example
Navigate to the Kubernetes documentation and search for "Ingress." Find an example that includes a `host` rule, as this is required by the lab.

### Step 2: Create and Modify the Ingress Manifest
Create a new YAML file (e.g., `my-ingress.yaml`) and paste the example code. Then, modify the key fields to match the lab's requirements.

**`my-ingress.yaml`:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "foobar.com"
    http:
      paths:
      - path: /foo-endpoint
        pathType: Prefix
        backend:
          service:
            name: my-cs
            port:
              number: 5678
```

**Key Modifications:**
-   `metadata.name`: Changed to a descriptive name, `my-ingress`.
-   `spec.rules[0].host`: Set to `"foobar.com"`.
-   `spec.rules[0].http.paths[0].path`: Set to `/foo-endpoint`.
-   `spec.rules[0].http.paths[0].backend.service.name`: Set to `my-cs`.
-   `spec.rules[0].http.paths[0].backend.service.port.number`: Set to `5678`.

### Step 3: Apply and Verify
Apply the manifest to create the Ingress resource and then use `kubectl describe` to verify that it has been configured correctly.

**Commands:**
```bash
# Apply the manifest
kubectl apply -f my-ingress.yaml

# Verify the Ingress resource
kubectl describe ingress my-ingress
```

**Verification Checklist:**
When you describe the Ingress, check the following fields:
-   **Name:** `my-ingress`
-   **Rules:**
    -   **Host:** `foobar.com`
    -   **Path:** `/foo-endpoint`
    -   **Backend:** `my-cs:5678`

## Exam Tip
Ingress controllers (like NGINX, Traefik, etc.) are a prerequisite for Ingress resources to function. While the exam environment will have this set up for you, it's important to remember that simply creating an Ingress resource does not automatically make it work in a real-world cluster without an Ingress controller running.

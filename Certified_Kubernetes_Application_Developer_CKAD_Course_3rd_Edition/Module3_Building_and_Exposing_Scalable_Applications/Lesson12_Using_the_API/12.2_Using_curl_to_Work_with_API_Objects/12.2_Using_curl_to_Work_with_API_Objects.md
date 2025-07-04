# 12.2 Using curl to Work with API Objects

## Introduction to curl with Kubernetes API

While `kubectl` is the primary tool for interacting with Kubernetes, sometimes you need to work directly with the API using `curl`. This approach is particularly useful for:

- Debugging API-related issues
- Understanding how `kubectl` works under the hood
- Scripting and automation
- Accessing the API from within containers where `kubectl` isn't available

## Setting Up kubectl proxy

Before making direct API calls, start the `kubectl proxy` to handle authentication:

```bash
# Start the proxy in the background
kubectl proxy --port=8001 &

# Verify it's running
jobs

# To stop the proxy later
kill %1
```

## Basic API Requests with curl

### Getting API Versions
```bash
# Get all API versions
curl http://localhost:8001/apis

# Get core API (v1) resources
curl http://localhost:8001/api/v1
```

### Working with Namespaced Resources

#### List Pods in Default Namespace
```bash
curl http://localhost:8001/api/v1/namespaces/default/pods
```

#### Get a Specific Pod
```bash
# First get the pod name
POD_NAME=$(kubectl get pods -o jsonpath='{.items[0].metadata.name}')

# Then query the specific pod
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME
```

### Working with Cluster-Scoped Resources

#### List Nodes
```bash
curl http://localhost:8001/api/v1/nodes
```

#### Get a Specific Node
```bash
# Get the first node name
NODE_NAME=$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')

# Query the node
curl http://localhost:8001/api/v1/nodes/$NODE_NAME
```

## Authentication and Headers

### Using Service Account Tokens
When running inside a pod, you can use the service account token:

```bash
# Get the service account token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Make an authenticated request
curl -H "Authorization: Bearer $TOKEN" \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### Setting Content-Type
For POST and PUT requests, set the correct content type:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"test"}}' \
  http://localhost:8001/api/v1/namespaces
```

## Common Operations

### Creating Resources
```bash
# Create a namespace
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"test"}}' \
  http://localhost:8001/api/v1/namespaces
```

### Updating Resources
```bash
# Annotate a pod
curl -X PATCH \
  -H "Content-Type: application/strategic-merge-patch+json" \
  -d '{"metadata":{"annotations":{"test":"value"}}}' \
  http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME
```

### Deleting Resources
```bash
# Delete a pod
curl -X DELETE http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME
```

## Advanced Queries

### Field Selectors
```bash
# Get pods on a specific node
curl "http://localhost:8001/api/v1/namespaces/default/pods?fieldSelector=spec.nodeName%3D$NODE_NAME"
```

### Label Selectors
```bash
# Get pods with a specific label
curl "http://localhost:8001/api/v1/namespaces/default/pods?labelSelector=app%3Dnginx"
```

### Watching Resources
```bash
# Watch for pod changes (supports long polling)
curl "http://localhost:8001/api/v1/namespaces/default/pods?watch=true"
```

## Error Handling

### Common HTTP Status Codes
- `200 OK`: Request succeeded
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid request format
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Not authorized to perform the action
- `404 Not Found`: Resource doesn't exist
- `409 Conflict`: Resource version conflict
- `422 Unprocessable Entity`: Invalid resource definition
- `500 Internal Server Error`: Server error

### Debugging Tips
```bash
# Get verbose output
curl -v http://localhost:8001/api/v1/namespaces/default/pods

# Get only the HTTP status code
curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/api/v1/namespaces/default/pods
```

## Security Considerations

### Minimize Use of Insecure Flags
Avoid using `--insecure` or `-k` in production. Instead:
- Use proper certificates
- Verify TLS connections
- Use service accounts with least privilege

### Secure Token Handling
- Never hardcode tokens in scripts
- Use Kubernetes secrets to manage sensitive data
- Set appropriate file permissions on token files

## Example: Complete Workflow

### 1. Create a Namespace
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"demo"}}' \
  http://localhost:8001/api/v1/namespaces
```

### 2. Create a Pod in the Namespace
```bash
cat > pod.json <<EOF
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx",
    "namespace": "demo",
    "labels": {
      "app": "nginx"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "nginx",
        "image": "nginx:alpine",
        "ports": [
          {
            "containerPort": 80
          }
        ]
      }
    ]
  }
}
EOF

curl -X POST \
  -H "Content-Type: application/json" \
  -d @pod.json \
  http://localhost:8001/api/v1/namespaces/demo/pods
```

### 3. Verify Pod Status
```bash
curl http://localhost:8001/api/v1/namespaces/demo/pods/nginx | jq '.status.phase'
```

### 4. Delete the Namespace (and all resources)
```bash
curl -X DELETE http://localhost:8001/api/v1/namespaces/demo
```

## Summary

Using `curl` with the Kubernetes API provides low-level access to cluster resources. While `kubectl` is more convenient for most operations, understanding how to work directly with the API is valuable for debugging, automation, and understanding Kubernetes internals. Always prioritize security by using proper authentication, authorization, and TLS when making API requests.

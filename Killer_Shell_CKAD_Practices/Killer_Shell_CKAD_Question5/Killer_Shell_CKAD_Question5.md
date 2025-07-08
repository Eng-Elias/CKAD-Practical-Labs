# CKAD Practice Question 5: Extracting and Decoding a Service Account Token

## Question
Team Neptune has a service account named `neptune-sa-v2` in the `neptune` namespace. You need to:

1. Find the secret associated with this service account
2. Extract the token from the secret
3. Decode the base64-encoded token
4. Save the decoded token to the file: `/root/neptune-sa-token.txt`

## Solution

### Step 1: Inspect the Service Account
```bash
# List service accounts in the neptune namespace
kubectl get serviceaccounts -n neptune

# Describe the specific service account
kubectl describe serviceaccount neptune-sa-v2 -n neptune
```

### Step 2: Find the Associated Secret
From the service account description, identify the secret name (e.g., `neptune-secret1`)

### Step 3: Extract and Decode the Token
```bash
# Get the token from the secret and decode it
kubectl get secret neptune-secret1 -n neptune -o jsonpath='{.data.token}' | base64 -d > /root/neptune-sa-token.txt

# Verify the token was saved correctly
cat /root/neptune-sa-token.txt
```

### Alternative Method (if you need to inspect the secret first)
```bash
# View all data in the secret
kubectl get secret neptune-secret1 -n neptune -o yaml

# Extract and decode just the token
echo "<base64-encoded-token>" | base64 -d > /root/neptune-sa-token.txt
```

## Explanation

### Key Concepts
1. **Service Accounts**: Used by pods to authenticate to the Kubernetes API server
2. **Secrets**: Store sensitive information like tokens, passwords, and keys
3. **Base64 Encoding**: Kubernetes stores secret data as base64-encoded strings
4. **Token Authentication**: Used for service-to-service authentication within the cluster

### Why This Solution Works
- `kubectl get secret` retrieves the secret data
- `-o jsonpath='{.data.token}'` extracts just the token field from the secret
- `base64 -d` decodes the base64-encoded token
- The output is redirected to the specified file

### Exam Tips
1. **Inspect Resources**: Always examine resources with `kubectl describe` to understand their structure
2. **JSONPath**: Learn basic JSONPath for extracting specific fields from JSON/YAML output
3. **Base64 Operations**: Remember that `base64 -d` decodes and `base64` encodes
4. **File Operations**: Be comfortable with file redirection (`>`) and appending (`>>`)
5. **Verification**: Always verify your operations (e.g., with `cat` or `ls -l`)

### Common Mistakes to Avoid
- Forgetting to specify the namespace with `-n`
- Not decoding the base64-encoded token
- Overwriting important files (use `>` carefully)
- Not verifying the output file contents
- Confusing service accounts with user accounts

## Additional Practice
1. Create a new service account and extract its token
2. Use the token to authenticate to the Kubernetes API
3. Create a pod that uses a specific service account
4. Inspect the mounted service account token in a running pod
5. Create custom secrets and extract data from them

## Related Commands
```bash
# Create a new service account
kubectl create serviceaccount my-sa -n my-namespace

# Get secrets in a namespace
kubectl get secrets -n my-namespace

# Describe a secret in detail
kubectl describe secret my-secret -n my-namespace

# Encode a string to base64
echo -n "my-secret-token" | base64

# Decode a base64 string
echo "bXktc2VjcmV0LXRva2VuCg==" | base64 -d

# Use jq for more complex JSON parsing
kubectl get secret my-secret -n my-namespace -o json | jq -r '.data.token' | base64 -d
```

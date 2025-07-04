# 14.5 Troubleshooting Authentication Problems

## Understanding Kubernetes Authentication

Kubernetes supports multiple authentication mechanisms:
1. **Client Certificates**
2. Bearer Tokens
3. Authentication Proxy
4. OpenID Connect (OIDC)
5. Webhook Token Authentication
6. Service Account Tokens

## Common Authentication Issues

### 1. Certificate Authentication Failures

**Symptoms:**
- `x509: certificate signed by unknown authority`
- `certificate has expired or is not yet valid`
- `tls: bad certificate`

**Troubleshooting:**

```bash
# Check certificate expiration
openssl x509 -in /path/to/cert.crt -noout -dates

# Verify certificate chain
openssl verify -CAfile /path/to/ca.crt /path/to/cert.crt

# Check kubeconfig certificate data
kubectl config view --raw -o jsonpath='{.users[?(@.name=="user-name")].user.client-certificate-data}' | base64 -d | openssl x509 -text

# Check API server certificate
openssl s_client -connect <api-server-ip>:6443 -showcerts </dev/null 2>/dev/null | openssl x509 -noout -text
```

### 2. Service Account Token Issues

**Symptoms:**
- `Unauthorized` errors
- `User "system:serviceaccount:..." cannot...`
- Token not found or invalid

**Troubleshooting:**

```bash
# Check if the service account exists
kubectl get serviceaccount <name> -n <namespace>

# Get the token secret name
kubectl get serviceaccount <name> -n <namespace> -o jsonpath='{.secrets[0].name}'

# View the token
kubectl get secret <token-secret-name> -n <namespace> -o jsonpath='{.data.token}' | base64 -d

# Check if the token is valid
kubectl --token=$(kubectl get secret <token-secret-name> -n <namespace> -o jsonpath='{.data.token}' | base64 -d) get pods
```

### 3. RBAC Permission Issues

**Symptoms:**
- `Forbidden` errors
- `User "..." cannot...`
- Permission denied on specific resources

**Troubleshooting:**

```bash
# Check user's permissions
kubectl auth can-i <verb> <resource> [--as=system:serviceaccount:<namespace>:<serviceaccount>]

# Example: Check if service account can list pods
kubectl auth can-i list pods --as=system:serviceaccount:default:my-serviceaccount

# Check roles and role bindings
kubectl get roles,rolebindings --all-namespaces
kubectl get clusterroles,clusterrolebindings

# Check what a specific role can do
kubectl describe role <role-name> -n <namespace>
kubectl describe clusterrole <clusterrole-name>

# Check what subjects are bound to a role
kubectl get rolebindings,clusterrolebindings --all-namespaces -o json | jq -r '.items[] | select(.roleRef.name=="<role-name>") | .subjects'
```

## Debugging Authentication

### 1. API Server Authentication Debugging

```bash
# Check API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Check authentication attempts with higher verbosity
kubectl logs -n kube-system kube-apiserver-<node-name> -v=4 | grep -i auth

# Check API server flags
ps aux | grep kube-apiserver
```

### 2. Authenticating as a Different User

```bash
# Create a temporary kubeconfig
kubectl config set-credentials temp-user --token=<token>
kubectl config set-context temp-context --cluster=<cluster-name> --user=temp-user
kubectl config use-context temp-context

# Test the new context
kubectl get pods
```

### 3. Checking Authentication Webhooks

```bash
# Check webhook configuration
kubectl get apiservices | grep authentication
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations

# Check webhook endpoints
kubectl get apiservice v1beta1.authentication.example.com -o yaml
```

## Common Authentication Scenarios

### 1. Service Account Not Found

**Issue:** `serviceaccount "my-sa" not found`

**Solution:**
```bash
# Create the service account
kubectl create serviceaccount my-sa -n <namespace>

# Check if the service account was created
kubectl get serviceaccount my-sa -n <namespace> -o yaml
```

### 2. Token Not Found

**Issue:** `secrets "my-sa-token-xxxxx" not found`

**Solution:**
```bash
# Delete the service account to regenerate the token
kubectl delete serviceaccount my-sa -n <namespace>
kubectl create serviceaccount my-sa -n <namespace>

# Or manually create a token
kubectl create token my-sa -n <namespace>
```

### 3. Certificate Expired

**Issue:** `certificate has expired or is not yet valid`

**Solution:**
```bash
# For kubeadm clusters
kubeadm alpha certs check-expiration
kubeadm alpha certs renew all

# For manual certificate management
# 1. Generate new certificates
# 2. Update kubeconfig
# 3. Restart API server
```

## Advanced Authentication Issues

### 1. OIDC Authentication

**Troubleshooting Steps:**

```bash
# Check OIDC configuration in API server
ps aux | grep oidc

# Verify OIDC provider is reachable
curl -s <oidc-issuer-url>/.well-known/openid-configuration

# Check for JWT token issues
jq -R 'split(".") | .[1] | @base64d | fromjson' <<< "<jwt-token>"
```

### 2. Webhook Authentication

**Troubleshooting Steps:**

```bash
# Check webhook configuration
kubectl get --raw /apis/authentication.k8s.io/v1/tokenreviews -X POST -H "Content-Type: application/json" -d '{"apiVersion": "authentication.k8s.io/v1", "kind": "TokenReview", "spec": {"token": "<token>"}}'

# Check webhook service
kubectl get svc -n <webhook-namespace>

# Check webhook endpoints
kubectl get endpoints <webhook-service> -n <webhook-namespace>
```

### 3. Certificate Authority Issues

**Troubleshooting Steps:**

```bash
# Check if the CA is trusted
openssl verify -CAfile /etc/kubernetes/pki/ca.crt /path/to/cert.crt

# Check certificate chain
openssl s_client -connect <api-server>:6443 -showcerts </dev/null 2>/dev/null | openssl x509 -noout -text

# Check API server CA bundle
kubectl get configmap -n kube-system extension-apiserver-authentication -o yaml
```

## Authentication Best Practices

1. **Use RBAC Effectively**
   - Follow the principle of least privilege
   - Use RoleBindings for namespace-scoped permissions
   - Use ClusterRoleBindings for cluster-scoped permissions

2. **Secure Service Accounts**
   - Avoid using default service account
   - Mount specific service account tokens
   - Use automountServiceAccountToken: false when not needed

3. **Certificate Management**
   - Rotate certificates regularly
   - Monitor certificate expiration
   - Use a certificate management solution (e.g., cert-manager)

4. **Audit Logging**
   - Enable audit logging for authentication attempts
   - Monitor for failed authentication attempts
   - Set up alerts for suspicious activities

## Common Error Messages and Solutions

### Error: `Unable to connect to the server: x509: certificate signed by unknown authority`

**Solution:**
```bash
# Check if using the correct kubeconfig
kubectl config view

# Check if the CA certificate is valid
openssl x509 -in ~/.kube/ca.crt -noout -text

# Update kubeconfig with correct CA
kubectl config set-cluster <cluster-name> --certificate-authority=/path/to/ca.crt --server=https://<api-server>:6443
```

### Error: `error: You must be logged in to the server (Unauthorized)`

**Solution:**
```bash
# Check current context
kubectl config current-context

# Check if the token is valid
kubectl --token=<token> get pods

# Check if the user has proper permissions
kubectl auth can-i list pods --as=system:serviceaccount:<namespace>:<serviceaccount>
```

### Error: `error: You must be logged in to the server (the server has asked for the client to provide credentials)`

**Solution:**
```bash
# Check kubeconfig credentials
kubectl config view

# Check if the token is valid
kubectl --token=<token> get pods

# Check if the certificate is valid
kubectl --client-certificate=/path/to/cert.crt --client-key=/path/to/key.key --certificate-authority=/path/to/ca.crt get pods
```

## Authentication Debugging Tools

### 1. kubectl auth

```bash
# Check if a user can perform an action
kubectl auth can-i <verb> <resource> [--as=user]

# Example: Check if current user can create deployments
kubectl auth can-i create deployments

# Check permissions for a service account
kubectl auth can-i list pods --as=system:serviceaccount:default:my-sa
```

### 2. kube-apiserver Audit Logs

```yaml
# Enable audit logging in kube-apiserver
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["pods"]
  - group: "apps"
    resources: ["deployments"]
  namespaces: ["default"]
  verbs: ["create", "update", "delete"]
```

### 3. RBAC Lookup

```bash
# Install rbac-lookup
kubectl krew install rbac-lookup

# Find permissions for a user
kubectl rbac-lookup <user>

# Find permissions for a service account
kubectl rbac-lookup <serviceaccount> -n <namespace>
```

## Conclusion

Troubleshooting authentication issues in Kubernetes requires a systematic approach. Start by identifying the authentication method in use, then check the relevant components and configurations. Always verify:

1. Certificates are valid and not expired
2. Service accounts exist and have proper tokens
3. RBAC rules are correctly configured
4. Authentication webhooks are functioning
5. Network connectivity to authentication services

By following the troubleshooting steps and best practices outlined in this guide, you can effectively diagnose and resolve authentication problems in your Kubernetes cluster.

# 11.8 Configuring the Docker Registry Access Secret

## Introduction to Docker Registry Secrets

Docker Registry Secrets in Kubernetes are used to store credentials for accessing private container registries. This is essential when your cluster needs to pull images from private repositories that require authentication.

## Creating Docker Registry Secrets

### 1. Using `kubectl create secret docker-registry`

```bash
# Basic syntax
kubectl create secret docker-registry <name> \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Example for Docker Hub
kubectl create secret docker-registry myregistrykey \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=myusername \
  --docker-password=mypassword \
  --docker-email=my.email@example.com

# Example for Google Container Registry (GCR)
kubectl create secret docker-registry gcr-json-key \
  --docker-server=https://gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat /path/to/service-account.json)" \
  --docker-email=any-email@example.com
```

### 2. Using YAML Manifest

```yaml
# registry-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
  namespace: default
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

To generate the base64-encoded docker config:

```bash
# Create docker config
DOCKER_CONFIG=$(cat <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "myusername",
      "password": "mypassword",
      "email": "my.email@example.com",
      "auth": "$(echo -n 'myusername:mypassword' | base64 -w0)"
    }
  }
}
EOF
)

# Encode to base64
echo $DOCKER_CONFIG | base64 -w0
```

## Using Registry Secrets in Pods

### 1. Using `imagePullSecrets` in Pod Spec

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg-pod
spec:
  containers:
  - name: private-reg-container
    image: private-registry.example.com/app:1.0
  imagePullSecrets:
  - name: myregistrykey
```

### 2. Using `imagePullSecrets` in ServiceAccount

A better approach is to associate the secret with a service account, so you don't need to specify it in every pod.

```yaml
# Create a service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
imagePullSecrets:
- name: myregistrykey
```

Then reference the service account in your pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  serviceAccountName: my-service-account  # Use the service account
  containers:
  - name: mycontainer
    image: private-registry.example.com/app:1.0
```

## Working with Different Registries

### 1. Docker Hub

```bash
kubectl create secret docker-registry docker-hub-creds \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email@example.com
```

### 2. Google Container Registry (GCR)

```bash
# Using JSON key file
gcloud iam service-accounts keys create key.json --iam-account=your-service-account@your-project.iam.gserviceaccount.com
kubectl create secret docker-registry gcr-json-key \
  --docker-server=https://gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat key.json)" \
  --docker-email=any-email@example.com
```

### 3. Amazon ECR

```bash
# Get ECR login command
AWS_ACCOUNT_ID=your-account-id
AWS_REGION=your-region
TOKEN=`aws ecr get-login-password --region $AWS_REGION`

# Create secret
kubectl create secret docker-registry ecr-credentials \
  --docker-server=https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$TOKEN \
  --docker-email=any-email@example.com
```

### 4. Azure Container Registry (ACR)

```bash
# Get ACR credentials
ACR_NAME=your-acr-name
SERVICE_PRINCIPAL_NAME=acr-service-principal
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
PASSWORD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query "password" --output tsv)
USER_NAME=$(az ad sp list --display-name $SERVICE_PRINCIPAL_NAME --query "[].appId" --output tsv)

# Create secret
kubectl create secret docker-registry acr-credentials \
  --docker-server=${ACR_NAME}.azurecr.io \
  --docker-username=$USER_NAME \
  --docker-password=$PASSWORD \
  --docker-email=any-email@example.com
```

## Best Practices

### 1. Use Namespaced Secrets
```yaml
# Create secret in a specific namespace
kubectl create secret docker-registry myregistrykey \
  --namespace=mynamespace \
  --docker-server=...
```

### 2. Rotate Credentials
- Regularly rotate registry credentials
- Update the secret and restart pods using it

### 3. Use Image Pull Policies
```yaml
spec:
  containers:
  - name: myapp
    image: private-registry.example.com/app:1.0
    imagePullPolicy: Always  # Options: Always, IfNotPresent, Never
```

### 4. Use Kubernetes Secrets for Registry Credentials
- Avoid using `.docker/config.json` in your image
- Use Kubernetes secrets for better security and management

## Troubleshooting

### 1. Image Pull Backoff
```bash
# Check pod status
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by='.metadata.creationTimestamp'
```

### 2. Inspect Pull Secrets
```bash
# List secrets
kubectl get secrets

# View secret details
kubectl describe secret myregistrykey

# Decode docker config
kubectl get secret myregistrykey -o jsonpath='{.data.\.dockerconfigjson}' | base64 --decode
```

### 3. Check Node Authentication
```bash
# SSH into node and test docker pull
docker login <registry-url> -u <username> -p <password>
docker pull <image>
```

## Example: Complete Deployment with Private Registry

### 1. Create Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
```

### 2. Create Registry Secret
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/dockerconfigjson
metadata:
  name: my-registry-key
  namespace: myapp
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

### 3. Create Service Account
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-service-account
  namespace: myapp
imagePullSecrets:
- name: my-registry-key
```

### 4. Create Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      serviceAccountName: myapp-service-account
      containers:
      - name: myapp
        image: private-registry.example.com/myapp:1.0.0
        ports:
        - containerPort: 8080
        imagePullPolicy: IfNotPresent
```

## Summary

Configuring Docker registry access in Kubernetes is essential for working with private container registries. By following the patterns and best practices outlined in this guide, you can securely manage container image pulls in your Kubernetes cluster.

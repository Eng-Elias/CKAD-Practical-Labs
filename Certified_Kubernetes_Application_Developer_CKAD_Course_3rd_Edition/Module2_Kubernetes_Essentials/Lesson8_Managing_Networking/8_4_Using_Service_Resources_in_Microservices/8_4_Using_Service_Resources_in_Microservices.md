# 8.4 Using Service Resources in Microservices

## Microservices Architecture with Kubernetes

### Core Concepts
- **Frontend Services**: User-facing components (e.g., web UI, API gateway)
- **Backend Services**: Business logic and data processing
- **Data Services**: Databases, caches, and message brokers

### Service Communication Patterns
1. **Synchronous**
   - HTTP/REST
   - gRPC
   - GraphQL

2. **Asynchronous**
   - Message queues (Kafka, RabbitMQ)
   - Event sourcing
   - CQRS

## Practical Example: Frontend-Backend Communication

### 1. Backend Service (Internal)

```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: backend
        image: my-backend:1.0
        ports:
        - containerPort: 3000
---
# backend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  labels:
    app: backend
    tier: backend
spec:
  type: ClusterIP
  selector:
    app: backend
    tier: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

### 2. Frontend Service (External)

```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: my-frontend:1.0
        ports:
        - containerPort: 80
        env:
        - name: BACKEND_SERVICE_URL
          value: "http://backend-service"
---
# frontend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  labels:
    app: frontend
    tier: frontend
spec:
  type: NodePort
  selector:
    app: frontend
    tier: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30007
```

## Service Discovery in Microservices

### Environment Variables
- Automatically injected into Pods
- Example: `BACKEND_SERVICE_SERVICE_HOST`
- Only available to Pods created after the Service

### DNS-Based Discovery
- Preferred method for service discovery
- Format: `<service-name>.<namespace>.svc.cluster.local`
- Example: `backend-service.default.svc.cluster.local`

## Advanced Patterns

### Service Mesh Integration
- **Istio**, **Linkerd**, or **Consul**
- Features:
  - Load balancing
  - Service-to-service authentication
  - Observability
  - Circuit breaking

### API Gateway Pattern
- Single entry point for all clients
- Request routing, composition, and protocol translation
- Authentication and authorization
- Rate limiting and monitoring

## Best Practices

### 1. Service Design
- Follow the Single Responsibility Principle
- Design for failure (implement retries, timeouts, circuit breakers)
- Version your APIs

### 2. Communication
- Use asynchronous communication when possible
- Implement proper timeouts and retries
- Use circuit breakers for fault tolerance

### 3. Observability
- Implement distributed tracing
- Use structured logging
- Monitor service health and performance

### 4. Security
- Use NetworkPolicies to restrict traffic
- Implement mutual TLS (mTLS) for service-to-service communication
- Use secrets for sensitive configuration

## Example: Complete Microservice

```yaml
# Complete example with ConfigMap and Secret
---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQxMjM=
---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
      tier: backend
  template:
    metadata:
      labels:
        app: user-service
        tier: backend
    spec:
      containers:
      - name: user-service
        image: user-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          value: "postgres-service"
        - name: DB_PORT
          value: "5432"
        - name: DB_NAME
          value: "users"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

## Next Steps
In the next section, we'll explore how DNS works in Kubernetes and how services are discovered using DNS.

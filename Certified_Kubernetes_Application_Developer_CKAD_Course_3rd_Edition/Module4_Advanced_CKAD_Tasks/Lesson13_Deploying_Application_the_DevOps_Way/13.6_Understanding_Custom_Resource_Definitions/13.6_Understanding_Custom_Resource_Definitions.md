# 13.6 Understanding Custom Resource Definitions (CRDs)

## Introduction to CRDs

Custom Resource Definitions (CRDs) extend the Kubernetes API by defining new resource types without modifying the Kubernetes source code. They allow users to create and manage custom resources just like built-in Kubernetes resources.

## Key Concepts

### What is a CRD?
- A way to extend the Kubernetes API
- Defines a new custom resource type
- Stored in etcd with the rest of the Kubernetes objects
- Managed through kubectl like native resources

### When to Use CRDs
- Packaging and deploying applications
- Managing external resources
- Implementing custom controllers/operators
- Extending Kubernetes functionality

## Creating a CRD

### Basic CRD Example

```yaml
# crd-backup.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                backupType:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
    shortNames:
    - bk
```

### Creating the CRD

```bash
# Create the CRD
kubectl apply -f crd-backup.yaml

# Verify the CRD was created
kubectl get crd
kubectl api-resources | grep backup
```

## Creating Custom Resources

### Example Custom Resource

```yaml
# backup-instance.yaml
apiVersion: stable.example.com/v1
kind: Backup
metadata:
  name: my-backup
spec:
  backupType: full
  image: my-backup-image:latest
  replicas: 2
```

### Creating and Managing the Custom Resource

```bash
# Create the custom resource
kubectl apply -f backup-instance.yaml

# List all backup resources
kubectl get backups

# Get details about a specific backup
kubectl describe backup my-backup

# Edit the resource
kubectl edit backup my-backup

# Delete the resource
kubectl delete backup my-backup
```

## CRD Validation

### Adding Validation to CRD

```yaml
# crd-with-validation.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                databaseName:
                  type: string
                  pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
                version:
                  type: string
                  enum: ["10", "11", "12"]
                storageGB:
                  type: integer
                  minimum: 1
                  maximum: 1000
                highAvailability:
                  type: boolean
              required: ["databaseName", "version"]
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

## CRD Status Subresource

### Adding Status Subresource

```yaml
# crd-with-status.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      subresources:
        status: {}
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                image:
                  type: string
                replicas:
                  type: integer
            status:
              type: object
              properties:
                availableReplicas:
                  type: integer
                conditions:
                  type: array
                  items:
                    type: object
                    properties:
                      type: {type: string}
                      status: {type: string}
                      message: {type: string}
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
```

## CRD Finalizers

### Adding Finalizers

```yaml
# crd-with-finalizer.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: resources.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                resourceName:
                  type: string
            finalizers:
              type: array
              items:
                type: string
  scope: Namespaced
  names:
    plural: resources
    singular: resource
    kind: Resource
```

## CRD Categories

### Adding Categories

```yaml
# crd-with-categories.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: cronjobs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                schedule:
                  type: string
                command:
                  type: string
  scope: Namespaced
  names:
    plural: cronjobs
    singular: cronjob
    kind: CronJob
    categories:
    - all
    - batch
```

## CRD Conversion

### Setting Up Conversion Webhooks

```yaml
# crd-with-conversion.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: configurations.stable.example.com
spec:
  group: stable.example.com
  conversion:
    strategy: Webhook
    webhook:
      clientConfig:
        service:
          namespace: default
          name: example-conversion-webhook-service
          path: /convert
      conversionReviewVersions: ["v1", "v1beta1"]
  versions:
  - name: v1
    served: true
    storage: true
    schema: {}
  - name: v1beta1
    served: true
    storage: false
    schema: {}
  scope: Namespaced
  names:
    plural: configurations
    singular: configuration
    kind: Configuration
```

## Best Practices for CRDs

### Naming Conventions
- Use a domain name you control for the group name
- Keep resource names consistent and descriptive
- Use camelCase for field names

### Versioning
- Start with v1alpha1 for experimental features
- Move to v1beta1 when the API is stable
- Use v1 for production-ready APIs

### Validation
- Always define schema validation
- Use the most specific type possible
- Add descriptions to document the purpose of fields

### Status and Conditions
- Use the status subresource
- Follow Kubernetes condition patterns
- Document the meaning of each condition

## Example: Complete CRD with All Features

```yaml
# complete-crd-example.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.stable.example.com
  annotations:
    "api-approved.kubernetes.io": "https://github.com/kubernetes/kubernetes/pull/12345"
spec:
  group: stable.example.com
  names:
    kind: Website
    listKind: WebsiteList
    plural: websites
    singular: website
    shortNames:
    - ws
    categories:
    - all
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    subresources:
      status: {}
      scale:
        specReplicasPath: .spec.replicas
        statusReplicasPath: .status.replicas
        labelSelectorPath: .status.labelSelector
    additionalPrinterColumns:
    - name: Replicas
      type: string
      description: The number of pods currently running
      jsonPath: .status.replicas
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              domain:
                type: string
                description: The domain name for the website
              replicas:
                type: integer
                description: Number of replicas to run
                minimum: 1
                maximum: 10
              containerPort:
                type: integer
                description: Port the container listens on
                default: 80
              resources:
                type: object
                properties:
                  limits:
                    type: object
                    properties:
                      cpu:
                        type: string
                        pattern: '^([0-9]+(\\.[0-9]+)?(m|K|M|G|T|P|E|Ki|Mi|Gi|Ti|Pi|Ei)?|0)$'
                      memory:
                        type: string
                        pattern: '^([0-9]+(\\.[0-9]+)?(m|K|M|G|T|P|E|Ki|Mi|Gi|Ti|Pi|Ei)?|0)$'
          status:
            type: object
            properties:
              replicas:
                type: integer
                description: The number of pods currently running
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type: {type: string}
                    status: {type: string}
                    lastTransitionTime: {type: string, format: date-time}
                    reason: {type: string}
                    message: {type: string}
  - name: v1beta1
    served: true
    storage: false
    # Conversion from v1beta1 to v1 would be handled by the webhook
    schema: {}
```

## Common Issues and Solutions

### Issue: CRD Validation Errors
```
The CustomResourceDefinition "example.stable.example.com" is invalid:
spec.validation.openAPIV3Schema.properties[spec].type: Required value: must not be empty for specified object fields
```
**Solution**: Ensure all nested objects in the schema have a type specified.

### Issue: CRD Version Mismatch
```
no matches for kind "Example" in version "stable.example.com/v1beta1"
```
**Solution**: Check that the version in the CRD definition matches the version in your resource YAML.

### Issue: CRD Not Ready
```
the server could not find the requested resource
```
**Solution**: Wait for the CRD to be established:
```bash
kubectl wait --for condition=established --timeout=60s crd/example.stable.example.com
```

## Tools for Working with CRDs

### 1. kubebuilder
```bash
# Install kubebuilder
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/

# Create a new project
kubebuilder init --domain example.com --repo github.com/example/operator
kubebuilder create api --group webapp --version v1 --kind Guestbook
```

### 2. Operator SDK
```bash
# Install operator-sdk
curl -L -o operator-sdk https://github.com/operator-framework/operator-sdk/releases/latest/download/operator-sdk_$(uname -s)_$(uname -m)
chmod +x operator-sdk && mv operator-sdk /usr/local/bin/

# Create a new operator
operator-sdk init --domain=example.com --repo=github.com/example/memcached-operator
operator-sdk create api --group=cache --version=v1alpha1 --kind=Memcached
```

## Advanced CRD Features

### 1. Defaulting Values
```yaml
# In the CRD schema
properties:
  spec:
    properties:
      port:
        type: integer
        default: 80
      enableTLS:
        type: boolean
        default: false
```

### 2. Immutable Fields
```yaml
# In the CRD schema
properties:
  spec:
    properties:
      domain:
        type: string
        x-kubernetes-immutable: true
```

### 3. Server-Side Apply
```yaml
# In the CRD schema
properties:
  metadata:
    type: object
    x-kubernetes-preserve-unknown-fields: true
  spec:
    type: object
    x-kubernetes-preserve-unknown-fields: true
```

## CRD Controller Example

Here's a simple controller pattern for a CRD:

```go
package main

import (
	"context"
	"fmt"
	"time"

	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/util/wait"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/util/workqueue"
	"k8s.io/klog"
)

type Controller struct {
	clientset     kubernetes.Interface
	informer      cache.SharedIndexInformer
	queue         workqueue.RateLimitingInterface
}

func (c *Controller) processNextItem() bool {
	// Get the next item from the queue
	key, quit := c.queue.Get()
	if quit {
		return false
	}
	// Mark the item as done processing
	defer c.queue.Done(key)

	// Process the item
	err := c.syncHandler(key.(string))
	if err == nil {
		// No error, reset the ratelimit counters
		c.queue.Forget(key)
	} else if c.queue.NumRequeues(key) < 5 {
		// Re-enqueue the key for processing
		c.queue.AddRateLimited(key)
	} else {
		// Too many retries, give up
		c.queue.Forget(key)
		klog.Errorf("Dropping key %q out of the queue: %v", key, err)
	}

	return true
}

func (c *Controller) syncHandler(key string) error {
	// Convert the namespace/name string into a distinct namespace and name
	namespace, name, err := cache.SplitMetaNamespaceKey(key)
	if err != nil {
		klog.Errorf("invalid resource key: %s", key)
		return nil
	}

	// Get the resource with this namespace/name
	obj, exists, err := c.informer.GetIndexer().GetByKey(key)
	if err != nil {
		return fmt.Errorf("error fetching object with key %s from store: %v", key, err)
	}

	if !exists {
		// Handle deletion
		klog.Infof("Custom resource %s has been deleted", key)
		return nil
	}

	// The custom resource exists, process it
	cr := obj.(*YourCustomResource)
	klog.Infof("Processing custom resource: %s/%s", namespace, name)

	// Your business logic here
	// For example, create/update deployments, services, etc.

	return nil
}

func (c *Controller) Run(workers int, stopCh <-chan struct{}) {
	defer c.queue.ShutDown()

	klog.Info("Starting custom resource controller")

	// Start the informer
	go c.informer.Run(stopCh)

	// Wait for the caches to sync
	if !cache.WaitForCacheSync(stopCh, c.informer.HasSynced) {
		klog.Error("timed out waiting for caches to sync")
		return
	}

	// Start worker threads
	for i := 0; i < workers; i++ {
		go wait.Until(c.runWorker, time.Second, stopCh)
	}

	// Block until the stop channel is closed
	<-stopCh
	klog.Info("Stopping custom resource controller")
}

func (c *Controller) runWorker() {
	for c.processNextItem() {
	}
}
```

This controller can be used to reconcile the state of your custom resources with the actual cluster state.

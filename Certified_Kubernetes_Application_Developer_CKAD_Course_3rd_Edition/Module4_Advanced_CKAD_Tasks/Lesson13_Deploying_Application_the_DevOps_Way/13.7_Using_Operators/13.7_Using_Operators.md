# 13.7 Using Operators

## Introduction to Kubernetes Operators

Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components. They follow Kubernetes principles, notably the control loop.

## What is a Kubernetes Operator?

- A method of packaging, deploying, and managing a Kubernetes application
- Extends the Kubernetes API
- Implements custom controllers to manage complex applications
- Encodes operational knowledge into software

## Operator Pattern

### Key Components
1. **Custom Resource Definitions (CRDs)**
   - Extend the Kubernetes API
   - Define custom resources

2. **Custom Controllers**
   - Watch the state of your cluster
   - Make or request changes where needed
   - Move the current state towards the desired state

3. **Operator Logic**
   - Contains the operational knowledge
   - Handles upgrades, backups, and failure recovery

## When to Use Operators

- When you need to automate complex application management
- For stateful applications that require cluster management
- When you need to encode operational knowledge
- For applications that require specific scaling or healing behaviors

## Building an Operator

### Prerequisites

1. Install the following tools:
   - Go (for writing the operator)
   - operator-sdk CLI
   - kubectl
   - Access to a Kubernetes cluster

### Step 1: Create a New Operator Project

```bash
# Create a new directory for your project
mkdir -p $GOPATH/src/github.com/example/memcached-operator
cd $GOPATH/src/github.com/example/memcached-operator

# Initialize a new operator using the operator-sdk
operator-sdk init --domain example.com --repo github.com/example/memcached-operator
```

### Step 2: Create a New API and Controller

```bash
# Create a new API for the Memcached application
operator-sdk create api --group cache --version v1alpha1 --kind Memcached --resource --controller
```

### Step 3: Define the API

Edit the `api/v1alpha1/memcached_types.go` file:

```go
package v1alpha1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// MemcachedSpec defines the desired state of Memcached
type MemcachedSpec struct {
	// Size is the size of the memcached deployment
	Size int32 `json:"size"`
	
	// Version is the version of Memcached to deploy
	Version string `json:"version,omitempty"`
}

// MemcachedStatus defines the observed state of Memcached
type MemcachedStatus struct {
	// Nodes are the names of the memcached pods
	Nodes []string `json:"nodes"`
}

//+kubebuilder:object:root=true
//+kubebuilder:subresource:status

// Memcached is the Schema for the memcacheds API
type Memcached struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   MemcachedSpec   `json:"spec,omitempty"`
	Status MemcachedStatus `json:"status,omitempty"`
}

//+kubebuilder:object:root=true

// MemcachedList contains a list of Memcached
type MemcachedList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata,omitempty"`
	Items           []Memcached `json:"items"`
}

func init() {
	SchemeBuilder.Register(&Memcached{}, &MemcachedList{})
}
```

### Step 4: Implement the Controller

Edit the `controllers/memcached_controller.go` file:

```go
package controllers

import (
	"context"
	"fmt"
	"reflect"

	"k8s.io/apimachinery/pkg/api/errors"
	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"

	cachev1alpha1 "github.com/example/memcached-operator/api/v1alpha1"
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/types"
	"k8s.io/apimachinery/pkg/util/intstr"
)

// MemcachedReconciler reconciles a Memcached object
type MemcachedReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=cache.example.com,resources=memcacheds,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=cache.example.com,resources=memcacheds/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=cache.example.com,resources=memcacheds/finalizers,verbs=update
//+kubebuilder:rbac:groups=apps,resources=deployments,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=core,resources=pods,verbs=get;list;

func (r *MemcachedReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := log.FromContext(ctx)

	// Fetch the Memcached instance
	memcached := &cachev1alpha1.Memcached{}
	err := r.Get(ctx, req.NamespacedName, memcached)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
			// Return and don't requeue
			log.Info("Memcached resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		// Error reading the object - requeue the request.
		log.Error(err, "Failed to get Memcached")
		return ctrl.Result{}, err
	}

	// Check if the deployment already exists, if not create a new one
	found := &appsv1.Deployment{}
	err = r.Get(ctx, types.NamespacedName{Name: memcached.Name, Namespace: memcached.Namespace}, found)
	if err != nil && errors.IsNotFound(err) {
		// Define a new deployment
		dep := r.deploymentForMemcached(memcached)
		log.Info("Creating a new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
		err = r.Create(ctx, dep)
		if err != nil {
			log.Error(err, "Failed to create new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
			return ctrl.Result{}, err
		}
		// Deployment created successfully - return and requeue
		return ctrl.Result{Requeue: true}, nil
	} else if err != nil {
		log.Error(err, "Failed to get Deployment")
		return ctrl.Result{}, err
	}

	// Ensure the deployment size is the same as the spec
	size := memcached.Spec.Size
	if *found.Spec.Replicas != size {
		found.Spec.Replicas = &size
		err = r.Update(ctx, found)
		if err != nil {
			log.Error(err, "Failed to update Deployment", "Deployment.Namespace", found.Namespace, "Deployment.Name", found.Name)
			return ctrl.Result{}, err
		}
		// Spec updated - return and requeue
		return ctrl.Result{Requeue: true}, nil
	}

	// Update the Memcached status with the pod names
	// List the pods for this memcached's deployment
	podList := &corev1.PodList{}
	listOpts := []client.ListOption{
		client.InNamespace(memcached.Namespace),
		client.MatchingLabels(labelsForMemcached(memcached.Name)),
	}
	if err = r.List(ctx, podList, listOpts...); err != nil {
		log.Error(err, "Failed to list pods", "Memcached.Namespace", memcached.Namespace, "Memcached.Name", memcached.Name)
		return ctrl.Result{}, err
	}
	podNames := getPodNames(podList.Items)

	// Update status.Nodes if needed
	if !reflect.DeepEqual(podNames, memcached.Status.Nodes) {
		memcached.Status.Nodes = podNames
		err := r.Status().Update(ctx, memcached)
		if err != nil {
			log.Error(err, "Failed to update Memcached status")
			return ctrl.Result{}, err
		}
	}

	return ctrl.Result{}, nil
}

// deploymentForMemcached returns a memcached Deployment object
func (r *MemcachedReconciler) deploymentForMemcached(m *cachev1alpha1.Memcached) *appsv1.Deployment {
	ls := labelsForMemcached(m.Name)
	replicas := m.Spec.Size

	dep := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      m.Name,
			Namespace: m.Namespace,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &replicas,
			Selector: &metav1.LabelSelector{
				MatchLabels: ls,
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: ls,
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{{
						Image:   "memcached:" + m.Spec.Version,
						Name:    "memcached",
						Command: []string{"memcached", "-m=64", "-o", "modern", "-v"},
						Ports: []corev1.ContainerPort{{
							ContainerPort: 11211,
							Name:          "memcached",
						}},
					}},
				},
			},
		},
	}

	// Set Memcached instance as the owner and controller
	ctrl.SetControllerReference(m, dep, r.Scheme)
	return dep
}

// labelsForMemcached returns the labels for selecting the resources
// belonging to the given memcached CR name.
func labelsForMemcached(name string) map[string]string {
	return map[string]string{"app": "memcached", "memcached_cr": name}
}

// getPodNames returns the pod names of the array of pods passed in
func getPodNames(pods []corev1.Pod) []string {
	var podNames []string
	for _, pod := range pods {
		podNames = append(podNames, pod.Name)
	}
	return podNames
}

// SetupWithManager sets up the controller with the Manager.
func (r *MemcachedReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&cachev1alpha1.Memcached{}).
		Owns(&appsv1.Deployment{}).
		Complete(r)
}
```

### Step 5: Install the CRDs

```bash
# Install the CRDs into the cluster
make install
```

### Step 6: Run the Operator

```bash
# Run the operator locally
make run

# Or build and deploy it to the cluster
make docker-build docker-push IMG=example.com/memcached-operator:v0.0.1
make deploy IMG=example.com/memcached-operator:v0.0.1
```

### Step 7: Create a Memcached Custom Resource

Create a sample Memcached resource:

```yaml
# config/samples/cache_v1alpha1_memcached.yaml
apiVersion: cache.example.com/v1alpha1
kind: Memcached
metadata:
  name: memcached-sample
spec:
  size: 3
  version: "1.6.9"
```

Apply it to your cluster:

```bash
kubectl apply -f config/samples/cache_v1alpha1_memcached.yaml
```

## Popular Operator Frameworks

### 1. Operator Framework
- **Website**: https://operatorframework.io/
- **Features**:
  - Operator SDK for building operators
  - Operator Lifecycle Manager (OLM) for managing operators
  - OperatorHub.io for discovering operators

### 2. Kubebuilder
- **Website**: https://book.kubebuilder.io/
- **Features**:
  - Framework for building Kubernetes APIs using CRDs
  - Uses controller-runtime library
  - Generates boilerplate code and manifests

### 3. KUDO (Kubernetes Universal Declarative Operator)
- **Website**: https://kudo.dev/
- **Features**:
  - Declarative approach to building operators
  - No need to write Go code
  - Supports complex, multi-resource applications

## Operator Best Practices

### 1. Idempotency
- Ensure your operator can handle being restarted at any time
- Make operations idempotent
- Handle conflicts gracefully

### 2. Event Handling
- Emit Kubernetes events for important operations
- Include detailed status in the custom resource
- Set appropriate conditions to indicate the state of the resource

### 3. Resource Management
- Set resource requests and limits
- Clean up resources when the CR is deleted
- Use finalizers for proper cleanup

### 4. Security
- Follow the principle of least privilege
- Define RBAC rules carefully
- Use security contexts in pods

### 5. Testing
- Write unit tests for your controllers
- Use integration tests with a test environment
- Consider end-to-end tests for complex workflows

## Example: Advanced Operator Features

### 1. Finalizers

```go
// Add finalizer to the Memcached CR
func (r *MemcachedReconciler) addFinalizer(memcached *cachev1alpha1.Memcached) error {
	if !containsString(memcached.GetFinalizers(), memcachedFinalizer) {
		memcached.SetFinalizers(append(memcached.GetFinalizers(), memcachedFinalizer))
		// Update CR
		err := r.Update(context.TODO(), memcached)
		if err != nil {
			return err
		}
	}
	return nil
}

// Handle finalizer
func (r *MemcachedReconciler) handleFinalizer(memcached *cachev1alpha1.Memcached) error {
	if memcached.GetDeletionTimestamp() != nil {
		if containsString(memcached.GetFinalizers(), memcachedFinalizer) {
			// Perform cleanup
			if err := r.deleteExternalResources(memcached); err != nil {
				return err
			}

			// Remove finalizer
			memcached.SetFinalizers(removeString(memcached.GetFinalizers(), memcachedFinalizer))
			if err := r.Update(context.TODO(), memcached); err != nil {
				return err
			}
		}
	}
	return nil
}
```

### 2. Status Conditions

```go
// Update status conditions
func (r *MemcachedReconciler) updateStatus(memcached *cachev1alpha1.Memcached, conditionType string, status corev1.ConditionStatus, reason, message string) error {
	now := metav1.Now()
	condition := cachev1alpha1.MemcachedCondition{
		Type:               conditionType,
		Status:             status,
		LastUpdateTime:     now,
		LastTransitionTime: now,
		Reason:             reason,
		Message:            message,
	}

	// Check if condition already exists
	conditionIndex := -1
	for i, cond := range memcached.Status.Conditions {
		if cond.Type == conditionType {
			conditionIndex = i
			break
		}
	}

	if conditionIndex >= 0 {
		// Update existing condition
		memcached.Status.Conditions[conditionIndex] = condition
	} else {
		// Add new condition
		memcached.Status.Conditions = append(memcached.Status.Conditions, condition)
	}

	// Update the status
	return r.Status().Update(context.TODO(), memcached)
}
```

### 3. Webhooks

#### Validation Webhook

```go
// +kubebuilder:webhook:path=/validate-cache-example-com-v1alpha1-memcached,mutating=false,failurePolicy=fail,groups=cache.example.com,resources=memcacheds,verbs=create;update,versions=v1alpha1,name=vmemcached.kb.io

var _ webhook.Validator = &Memcached{}

// ValidateCreate implements webhook.Validator so a webhook will be registered for the type
func (r *Memcached) ValidateCreate() error {
	memcachedlog.Info("validate create", "name", r.Name)

	return r.validateMemcached()
}

// ValidateUpdate implements webhook.Validator so a webhook will be registered for the type
func (r *Memcached) ValidateUpdate(old runtime.Object) error {
	memcachedlog.Info("validate update", "name", r.Name)

	return r.validateMemcached()
}

// ValidateDelete implements webhook.Validator so a webhook will be registered for the type
func (r *Memcached) ValidateDelete() error {
	memcachedlog.Info("validate delete", "name", r.Name)

	// No validation on delete
	return nil
}

func (r *Memcached) validateMemcached() error {
	var allErrs field.ErrorList
	
	if r.Spec.Size <= 0 {
		allErrs = append(allErrs, field.Invalid(
			field.NewPath("spec").Child("size"),
			r.Spec.Size,
			"size must be greater than 0",
		))
	}
	
	if r.Spec.Version == "" {
		allErrs = append(allErrs, field.Required(
			field.NewPath("spec").Child("version"),
			"version must be specified",
		))
	}
	
	if len(allErrs) == 0 {
		return nil
	}
	
	return apierrors.NewInvalid(
		schema.GroupKind{Group: "cache.example.com", Kind: "Memcached"},
		r.Name, allErrs)
}
```

## Operator Lifecycle Manager (OLM)

OLM helps manage operators and their dependencies in a cluster.

### Install OLM

```bash
# Install OLM
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.20.0/install.sh | bash -s v0.20.0

# Verify installation
kubectl get pods -n olm
```

### Create an Operator Bundle

```bash
# Create a bundle directory
mkdir -p bundle/manifests

# Copy CRDs and CSV
cp config/crd/bases/*.yaml bundle/manifests/
cp config/manifests/bases/memcached-operator.clusterserviceversion.yaml bundle/manifests/

# Generate bundle
operator-sdk bundle create quay.io/example/memcached-operator-bundle:v0.1.0 --directory bundle --package memcached-operator --channels alpha --default-channel alpha

# Push the bundle
docker push quay.io/example/memcached-operator-bundle:v0.1.0
```

### Deploy the Operator with OLM

```bash
# Create an OperatorGroup
cat <<EOF | kubectl apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: memcached-operator-group
  namespace: default
spec:
  targetNamespaces:
  - default
---
# Create a Subscription
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: memcached-operator-subscription
  namespace: default
spec:
  channel: alpha
  name: memcached-operator
  source: operatorhubio-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
  startingCSV: memcached-operator.v0.1.0
EOF
```

## Monitoring and Logging

### Prometheus Metrics

```go
import (
	"sigs.k8s.io/controller-runtime/pkg/metrics"
	"github.com/prometheus/client_golang/prometheus"
)

var (
	reconcileCounter = prometheus.NewCounter(
		prometheus.CounterOpts{
			Name: "memcached_controller_reconcile_total",
			Help: "Number of reconcile operations",
		},
	)
)

func init() {
	// Register metrics with the global prometheus registry
	metrics.Registry.MustRegister(reconcileCounter)
}

// In your Reconcile method
func (r *MemcachedReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	reconcileCounter.Inc()
	// ... rest of the reconcile logic
}
```

### Structured Logging

```go
import (
	"sigs.k8s.io/controller-runtime/pkg/log"
)

func (r *MemcachedReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := log.FromContext(ctx)
	
	log.Info("Starting reconcile loop", "request", req.NamespacedName)
	
	// Your reconcile logic here
	
	log.Info("Finished reconcile", "result", result)
	return result, nil
}
```

## Testing Operators

### Unit Tests

```go
func TestMemcachedReconciler_Reconcile(t *testing.T) {
	// Create a test Memcached object
	memcached := &cachev1alpha1.Memcached{
		ObjectMeta: metav1.ObjectMeta{
			Name:      "test-memcached",
			Namespace: "default",
		},
		Spec: cachev1alpha1.MemcachedSpec{
			Size:    3,
			Version: "1.6.9",
		},
	}

	// Register operator types with the scheme
	s := scheme.Scheme
	s.AddKnownTypes(cachev1alpha1.GroupVersion, memcached)

	// Create a fake client to mock API calls
	cl := fake.NewClientBuilder().WithScheme(s).WithObjects(memcached).Build()

	// Create a ReconcileMemcached object with the fake client
	r := &MemcachedReconciler{
		Client: cl,
		Scheme: s,
	}

	// Mock request to simulate Reconcile() being called
	req := reconcile.Request{
		NamespacedName: types.NamespacedName{
			Name:      "test-memcached",
			Namespace: "default",
		},
	}

	// Call Reconcile()
	res, err := r.Reconcile(context.TODO(), req)
	if err != nil {
		t.Fatalf("reconcile: (%v)", err)
	}

	// Check the result
	if res.Requeue {
		t.Error("reconcile did not requeue request as expected")
	}

	// Check if deployment has been created
	dep := &appsv1.Deployment{}
	err = cl.Get(context.TODO(), req.NamespacedName, dep)
	if err != nil {
		t.Fatalf("get deployment: (%v)", err)
	}

	// Check the number of replicas
	size := *dep.Spec.Replicas
	if size != 3 {
		t.Errorf("dep size (%d) is not the expected size (%d)", size, 3)
	}
}
```

### Integration Tests

```go
var _ = Describe("Memcached controller", func() {
	Context("When creating a Memcached resource", func() {
		It("Should create a deployment with the specified number of replicas", func() {
			By("By creating a new Memcached")
			ctx := context.Background()
			
			memcached := &cachev1alpha1.Memcached{
				TypeMeta: metav1.TypeMeta{
					APIVersion: "cache.example.com/v1alpha1",
					Kind:       "Memcached",
				},
				ObjectMeta: metav1.ObjectMeta{
					Name:      "test-memcached",
					Namespace: "default",
				},
				Spec: cachev1alpha1.MemcachedSpec{
					Size:    3,
					Version: "1.6.9",
				},
			}
			
			Expect(k8sClient.Create(ctx, memcached)).Should(Succeed())
			
			// Verify the deployment is created
			deployment := &appsv1.Deployment{}
			Eventually(func() bool {
				err := k8sClient.Get(ctx, types.NamespacedName{Name: "test-memcached", Namespace: "default"}, deployment)
				return err == nil
			}, timeout, interval).Should(BeTrue())
			
			// Verify the number of replicas
			Expect(*deployment.Spec.Replicas).To(Equal(int32(3)))
		})
	})
})
```

## Operator Maturity Model

### Level 1: Basic Install
- CRD is installed
- Basic deployment of the application
- No upgrade or failure recovery

### Level 2: Seamless Upgrades
- Support for in-place upgrades
- Handle version skew
- Backup before upgrade

### Level 3: Full Lifecycle
- Handle all application lifecycle events
- Backup and restore
- Failure recovery
- Auto-scaling

### Level 4: Deep Insights
- Application metrics
- Log aggregation
- Alerting
- Usage reporting

### Level 5: Auto-pilot
- Auto-scaling based on metrics
- Auto-healing
- Predictive scaling
- Cost optimization

## Conclusion

Kubernetes Operators are a powerful pattern for managing complex applications on Kubernetes. By following the operator pattern, you can encode operational knowledge into software, making your applications more reliable and easier to manage. Start with the basics, follow best practices, and gradually incorporate more advanced features as your needs grow.

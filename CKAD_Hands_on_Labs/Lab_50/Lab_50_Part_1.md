# CKAD Lab 55 (Part 1): Using `kubectl explain` to Build a `nodeAffinity` Rule

## Objective
This two-part lab demonstrates the most critical skill for the CKAD exam: using `kubectl explain` to discover the correct YAML structure for a complex feature like `nodeAffinity` when you don't have an example manifest.

## The Challenge
Create a `Deployment` that uses `nodeAffinity` to ensure its pods are only scheduled on nodes that have the label `disk=ssd`.

## Part 1 Solution: The Discovery Process
We start with a basic deployment manifest and use `kubectl explain` to find where and how to add the `nodeAffinity` rule.

### Step 1: The Starting Point
For a `Deployment`, all pod-related specifications, including scheduling rules, go under `spec.template.spec`.

### Step 2: The `kubectl explain` Drill-Down
We will navigate the object schema step-by-step.

**1. Start at the Pod Template Spec:**
```bash
kubectl explain deployment.spec.template.spec
```
Looking at the output, we see a field named `affinity`.

**2. Explore `affinity`:**
```bash
kubectl explain deployment.spec.template.spec.affinity
```
This shows us `nodeAffinity` and `podAffinity`. We need `nodeAffinity`.

**3. Explore `nodeAffinity`:**
```bash
kubectl explain deployment.spec.template.spec.affinity.nodeAffinity
```
This reveals two main types of rules:
-   `requiredDuringSchedulingIgnoredDuringExecution`: Hard requirement. The pod **will not** be scheduled unless the rule is met.
-   `preferredDuringSchedulingIgnoredDuringExecution`: Soft requirement. The scheduler will **try** to enforce the rule but will schedule the pod anyway if it can't be met.

We'll use the `required` rule.

**4. Explore the `required` Rule:**
```bash
kubectl explain deployment.spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution
```
This shows a field called `nodeSelectorTerms`, which is a list of selectors.

**5. Explore `nodeSelectorTerms`:**
```bash
kubectl explain deployment.spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms
```
This shows `matchExpressions` and `matchFields`. We need to match labels, so we use `matchExpressions`.

**6. Explore `matchExpressions`:**
```bash
kubectl explain deployment.spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions
```
Finally, this reveals the fields we need to build our rule:
-   `key`: The label key to match (e.g., `disk`).
-   `operator`: The logical operator (e.g., `In`, `NotIn`, `Exists`).
-   `values`: A list of label values to match against.

## Next Steps
We have successfully discovered the complete path and the required fields. In Part 2, we will use this information to construct the final YAML manifest for the deployment with the correct `nodeAffinity` block.

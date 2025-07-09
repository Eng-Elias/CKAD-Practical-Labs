# CKAD Practice Question 22: Working with Labels and Annotations

## Question
Team Sunny needs to manage their pods in the `sun` namespace. Your tasks are:

1. **Add a new label** `protected=true` to all pods that have either:
   - Label `type=worker` OR
   - Label `type=runner`

2. **Add an annotation** `protected="do not delete this pod"` to all pods that have the new `protected=true` label.

**Requirements**:
- Work in the `sun` namespace
- Don't delete any existing labels or annotations
- Verify your changes after making them

## Solution

### Step 1: View Current Pods and Labels
```bash
# List all pods with their labels
kubectl -n sun get pods --show-labels

# View more details about the pods
kubectl -n sun describe pods
```

### Step 2: Add Label to Worker Pods
```bash
# Add protected=true to pods with type=worker
kubectl -n sun label pods --selector type=worker protected=true --overwrite
```

### Step 3: Add Label to Runner Pods
```bash
# Add protected=true to pods with type=runner
kubectl -n sun label pods --selector type=runner protected=true --overwrite
```

### Step 4: Add Annotation to Protected Pods
```bash
# Add annotation to all pods with protected=true
kubectl -n sun annotate pods --selector protected=true protected="do not delete this pod" --overwrite
```

### Step 5: Verify the Changes
```bash
# Check labels for all pods
kubectl -n sun get pods --show-labels

# Check annotations for protected pods
kubectl -n sun get pods -l protected=true -o jsonpath='{range .items[*]}{.metadata.name}: {.metadata.annotations}{"\n"}{end}'

# Alternative: View all details of a pod
kubectl -n sun describe pod <pod-name>
```

## Explanation

### Key Concepts
1. **Labels**:
   - Key-value pairs used to identify and select objects
   - Can be used for filtering and grouping
   - Example: `type=worker`, `environment=production`

2. **Annotations**:
   - Store non-identifying metadata
   - Used for tooling, libraries, etc.
   - Example: `description="This pod runs the web server"`

3. **Selectors**:
   - Used to select objects based on their labels
   - Supports equality-based (`=`, `==`, `!=`) and set-based (`in`, `notin`, `exists`) requirements

### Why This Solution Works
- Uses `kubectl label` to add/update labels
- Uses `kubectl annotate` to add/update annotations
- The `--selector` flag targets specific pods
- `--overwrite` ensures existing values are updated
- Verification commands confirm the changes were applied

### Exam Tips
1. **Label Management**:
   - Use `kubectl label` to add/update labels
   - Use `--overwrite` to update existing labels
   - Use `--selector` to target specific resources

2. **Annotation Management**:
   - Use `kubectl annotate` to add/update annotations
   - Annotations are useful for metadata that's not used for selection

3. **Verification**:
   - Always verify your changes
   - Use `--show-labels` to check labels
   - Use `-o jsonpath` for custom output formatting

### Common Mistakes to Avoid
- Forgetting to specify the namespace
- Not using `--overwrite` when updating existing labels/annotations
- Incorrect selector syntax
- Not verifying the changes were applied
- Using labels when annotations would be more appropriate

## Additional Practice
1. Remove a specific label from pods
2. Add multiple labels at once
3. Use set-based selectors
4. Create a deployment with custom labels
5. Use labels with other resources (services, deployments, etc.)

## Related Commands
```bash
# Remove a label
kubectl -n <namespace> label pods -l <label> <label-key>-

# Add multiple labels
kubectl -n <namespace> label pods <pod-name> <key1>=<value1> <key2>=<value2>

# View specific label
kubectl -n <namespace> get pods -L <label-key>

# Filter using multiple labels
kubectl -n <namespace> get pods -l '<key1>=<value1>,<key2>=<value2>'

# Remove an annotation
kubectl -n <namespace> annotate pods <pod-name> <annotation-key>-
```

# CKAD Lab 57 (Part 7): Applying and Troubleshooting the Manifest

## Objective
With the complete `Deployment` manifest assembled, the next step is to apply it to the cluster and fix any errors that arise. This is a critical skill for the CKAD exam, as it's rare to write a complex manifest perfectly on the first try.

## Part 7 Solution: Iterative Troubleshooting

### Error 1: `did not find expected key` (Indentation)
**Symptom:** The API server reports an error about a missing key, often pointing to the line where a block is defined (e.g., `resources`).

**Cause:** This is almost always an indentation error. In this case, the `resources` block was not correctly aligned with other container properties like `image` and `name`.

**Fix:** Correct the indentation so that `resources` is a direct child of the `containers` array element.

### Error 2: `unknown field "Operator"` (Case Sensitivity)
**Symptom:** The API server rejects the manifest, complaining about an unknown field.

**Cause:** Kubernetes API fields are case-sensitive. The `nodeAffinity` operator was written as `Operator` instead of the correct `operator`.

**Fix:** Change `Operator: In` to `operator: In`.

### Error 3: `selector does not match template labels`
**Symptom:** The `Deployment` is rejected with a validation error stating the selector and template labels do not match.

**Cause:** This is a fundamental rule of `Deployments` (and `ReplicaSets`, `StatefulSets`, etc.). The set of labels defined in `spec.selector.matchLabels` must be a subset of (and usually identical to) the labels defined in `spec.template.metadata.labels`. This is how the controller knows which pods belong to it.

**Fix:** Ensure the key-value pairs in both sections are identical.

## Next Steps
With these errors identified, Part 8 will apply the final, corrected manifest and perform a full verification to ensure every requirement of this complex lab has been met.

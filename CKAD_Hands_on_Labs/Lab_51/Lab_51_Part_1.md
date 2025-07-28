# CKAD Lab 51 (Part 1): Reinforcing `nodeAffinity` with `kubectl explain`

## Objective
This lab reinforces the essential skill of using `kubectl explain` to build a `Deployment` manifest with a `nodeAffinity` rule. Mastering this discovery process is key to solving complex scheduling challenges on the CKAD exam without relying on pre-existing examples.

## The Challenge
1.  Label a node with the key-value pair `foo=bar`.
2.  Create a `Deployment` named `foo-deployment` with 5 replicas.
3.  Add a `nodeAffinity` rule to ensure the deployment's pods are only scheduled on the node labeled `foo=bar`.

## Part 1 Solution: The Discovery Process Revisited
As with Lab 55, we begin with a basic deployment manifest and use `kubectl explain` to find the correct structure for the `nodeAffinity` rule. For a `Deployment`, all pod-related specs are nested under `spec.template.spec`.

### The `kubectl explain` Path
The discovery path is the same, and it's worth committing to memory:

1.  **Start Point:** `kubectl explain deployment.spec.template.spec`
    -   Find `affinity`.

2.  **Next Step:** `kubectl explain ...affinity`
    -   Find `nodeAffinity`.

3.  **Next Step:** `kubectl explain ...nodeAffinity`
    -   Choose a rule type, for example, `requiredDuringSchedulingIgnoredDuringExecution` for a hard requirement.

4.  **Next Step:** `kubectl explain ...requiredDuringSchedulingIgnoredDuringExecution`
    -   Find `nodeSelectorTerms`.

5.  **Next Step:** `kubectl explain ...nodeSelectorTerms`
    -   Find `matchExpressions`.

6.  **End Point:** `kubectl explain ...matchExpressions`
    -   This reveals the final set of keys needed to build the rule: `key`, `operator`, and `values`.

This systematic approach removes all guesswork from writing the manifest.

## Next Steps
Having re-confirmed the required fields and their hierarchy, Part 2 will focus on constructing the final YAML. We will again see how the API server's error messages guide us in correctly structuring the `nodeSelectorTerms` and `matchExpressions` as arrays of objects.

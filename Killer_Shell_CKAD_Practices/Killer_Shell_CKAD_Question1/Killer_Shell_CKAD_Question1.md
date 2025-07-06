# CKAD Practice Question 1: List All Namespaces

## Question
**Task:** The DevOps team would like to get a list of all namespaces in the cluster. Get the list and save it to the file `/opt/course/1/namespaces`.

## Context
- You are working in a Kubernetes cluster environment
- You have `kubectl` configured with the necessary permissions
- The alias `k` is available for `kubectl`

## Solution

### Step 1: List all namespaces
```bash
kubectl get namespaces
```

### Step 2: Save the output to the specified file
```bash
kubectl get namespaces > /opt/course/1/namespaces
```

### Step 3: Verify the file content
```bash
cat /opt/course/1/namespaces
```

## Explanation

### Key Concepts
1. **Namespaces**: Logical partitions in a Kubernetes cluster that help organize and separate resources.
2. **kubectl get**: Command to list resources in the cluster.
3. **Output Redirection**: The `>` operator is used to redirect command output to a file.

### Why This Works
- `kubectl get namespaces` retrieves a list of all namespaces in the cluster
- The `>` operator redirects the standard output to the specified file
- The file is created if it doesn't exist, or overwritten if it does

### Exam Tips
1. **Use Aliases**: In the exam, `k` is aliased to `kubectl` to save time.
2. **Verify Your Work**: Always verify the output file after creation.
3. **File Paths**: Pay close attention to the exact file path specified in the question.
4. **Time Management**: Simple questions like this should be completed quickly to save time for more complex tasks.

### Common Mistakes to Avoid
- Forgetting to include the `s` in `namespaces`
- Using incorrect file paths
- Not verifying the output file

## Additional Practice
Try these variations to solidify your understanding:
1. List namespaces in a specific output format (e.g., JSON or YAML)
2. Save the output in a different format
3. Append to an existing file instead of overwriting

## Related Commands
```bash
# List namespaces in JSON format
kubectl get namespaces -o json

# List namespaces in YAML format
kubectl get namespaces -o yaml

# Append to an existing file
kubectl get namespaces >> existing_file.txt
```

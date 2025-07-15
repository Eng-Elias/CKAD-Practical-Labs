# CKAD Lab 24: Pod with Two Containers (Sleep and Environment Variable)

## Objective
Create a single pod with two containers, each with a specific requirement:
1.  **Container 1:** Must run a process that sleeps for one hour (3600 seconds).
2.  **Container 2:** Must have an environment variable `GREETING` set to the value `"Hello World"`.

This lab simulates a typical CKAD exam question where you must combine multiple concepts into a single Kubernetes resource.

## Solution Strategy
Instead of writing the YAML from scratch, we will build it piece by piece using examples, just as you would in the exam.
1.  Generate a basic pod manifest.
2.  Find an example for a container `command` to make it sleep.
3.  Find an example for defining `env` (environment variables).
4.  Combine these pieces into a final manifest.

## Solution

### Step 1: Generate the Base Manifest
Start by creating the YAML for a simple, single-container pod.

**Command:**
```bash
kubectl run my-app --image=nginx --dry-run=client -o yaml > my-app-pod.yaml
```

### Step 2: Modify the First Container to Sleep
Edit `my-app-pod.yaml`. For the first container, we need to add a `command` that overrides the default container command and tells it to sleep. We'll also change its name for clarity.

**YAML Snippet for `command`:**
```yaml
    command: ['sh', '-c', 'sleep 3600']
```

### Step 3: Add and Modify the Second Container for Environment Variables
Copy the entire first container's definition and paste it as a second item in the `containers` array. Change its name to `env-container`. Then, add the `env` section to define the required environment variable.

**YAML Snippet for `env`:**
```yaml
    env:
    - name: GREETING
      value: "Hello World"
```

### Step 4: Assemble the Final Manifest
Combine all the pieces into the final `my-app-pod.yaml` file.

**`my-app-pod.yaml`:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-app
  name: my-app
spec:
  containers:
  # --- Container 1: Sleeps for an hour ---
  - image: busybox # busybox is a lightweight image perfect for this
    name: sleep-container
    # This command keeps the container running
    command: ['sh', '-c', 'sleep 3600']
    resources: {}

  # --- Container 2: Has an environment variable ---
  - image: nginx
    name: env-container
    # This section defines the environment variable
    env:
    - name: GREETING
      value: "Hello World"
    resources: {}

  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

### Step 5: Apply and Verify
Create the pod and use `kubectl describe` to verify that both containers are configured correctly.

**Commands:**
```bash
# Create the pod
kubectl apply -f my-app-pod.yaml

# Wait for it to be ready (2/2)
kubectl get pod my-app

# Describe the pod to check the details
kubectl describe pod my-app
```

**Verification Checklist:**
-   Under the `Containers` section, you should see both `sleep-container` and `env-container`.
-   `sleep-container` should have the `command: ['sh', '-c', 'sleep 3600']`.
-   `env-container` should have the `Environment` section with `GREETING: Hello World`.

## Exam Tip
This lab is a perfect representation of a CKAD task. You are rarely asked to create a simple, default resource. You will almost always need to customize it with specific fields like `command`, `args`, `env`, `volumeMounts`, or `ports`. Practice finding these fields in the Kubernetes documentation and integrating them into a base YAML template.

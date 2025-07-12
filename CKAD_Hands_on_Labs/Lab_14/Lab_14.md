# CKAD Lab 14: Shelling into Running Pods and Containers

## Objective
Learn how to use `kubectl exec` to gain an interactive shell inside a running container. This is a crucial skill for live debugging, inspecting the container's filesystem, and checking its running processes.

## The `kubectl exec` Command
The `kubectl exec` command allows you to execute a command inside a container. When used with the `-i` (interactive) and `-t` (TTY) flags, it can provide you with a fully interactive shell session.

**The Standard Pattern:**
```bash
kubectl exec -it <pod-name> -- <command-to-run>
```
-   `-i` or `--stdin`: Keep STDIN open, allowing you to type commands.
-   `-t` or `--tty`: Allocate a pseudo-TTY, which makes the session interactive.
-   `--`: This double-dash separates the `kubectl` arguments from the command you want to run inside the container. It's a best practice to ensure your command is interpreted correctly.

## Solution
This lab demonstrates how to shell into two different types of containers: one based on `busybox` (which has `sh`) and one based on `nginx` (which has `bash`).

### Step 1: Create Pods to Work With
First, create a couple of pods to practice on.

**Commands:**
```bash
# Create a simple busybox pod that sleeps forever
kubectl run busybox --image=busybox --command -- sleep 3600

# Create a standard nginx pod
kubectl run nginx --image=nginx
```

### Step 2: Shell into the `busybox` Pod
The `busybox` image is very minimal and only includes the Bourne shell (`sh`).

**Command:**
```bash
kubectl exec -it busybox -- sh
```
Once inside, you are at the container's command prompt. You can run commands like `ls`, `hostname`, or `ping`.
```bash
/ # hostname
busybox
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # exit
```

### Step 3: Shell into the `nginx` Pod
The standard `nginx` image includes the more feature-rich `bash` shell.

**Command:**
```bash
kubectl exec -it nginx -- bash
```
Again, you are now inside the `nginx` container. Note that different images come with different tools. For example, the `nginx` image may not have `ping`, but it might have `curl`.
```bash
root@nginx:/# hostname
nginx
root@nginx:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
root@nginx:/# exit
```

### Step 4: Shelling into a Specific Container
If your pod has multiple containers, you must use the `-c <container-name>` flag to specify which one you want to `exec` into.

**Command:**
```bash
# For a pod named 'my-pod' with containers 'app' and 'sidecar'
kubectl exec -it my-pod -c app -- bash
```

## Explanation
`kubectl exec` is an indispensable tool for interactive debugging. It gives you a direct look inside a running container's environment, allowing you to:
-   Check the contents of configuration files mounted as volumes.
-   Verify network connectivity from within the pod.
-   Inspect running processes.
-   Check environment variables.

Remember that the available shell and tools depend entirely on what is included in the container image.

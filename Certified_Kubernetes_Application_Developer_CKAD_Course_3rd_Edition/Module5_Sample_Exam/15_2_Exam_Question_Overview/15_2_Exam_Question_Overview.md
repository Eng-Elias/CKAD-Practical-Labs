# 15.2 Exam Question Overview

## Introduction

If you are almost ready for the exam, try to do the exam in two hours. If this is your first time, take the time you need.

--- 

### 1. Namespaces and Pods

*   Create a namespace named `ckad-ns1`.
*   In this namespace, run the following pods:
    *   A pod named `pod-a` running the `httpd` server image.
    *   A pod named `pod-b` running the `nginx` server image and the `alpine` image.

### 2. Secrets and Deployments

*   Create a secret that defines the variable `password=secret`.
*   Create a deployment named `secret-app` which starts the `nginx` image and uses this variable.

### 3. Dockerfile and Images

*   Create a `Dockerfile` that runs an `alpine` image with the command `echo hello world` as the default command.
*   Build the image.
*   Export it into OCI format to a tar file named `greet-world.tar`.

### 4. Multi-container Pods (Sidecar)

*   Create a multi-container pod named `sidecar-pod` that runs in the `ckad-ns3` namespace.
*   The primary container runs `busybox` and writes the output of the `date` command to the `/var/log/date.log` file every 5 seconds.
*   The second container should run as a sidecar and provide `nginx` web access to this file using a `hostPath` shared volume.
*   Mount this on `user/share/nginx/html`.
*   Make sure that the image for this container is only pulled if it is not available on the local system yet.

### 5. Fixing Deployments

*   Start the deployment from the `redis.yaml` file in the course git repository.
*   Fix any problems that may occur while starting it.

### 6. Probes

*   Create a pod that runs the `nginx` web server, offering its services on port 80 and running in the `ckad-ns3` namespace.
*   The pod should check the `healthz` path on the API server before starting the main container.

### 7. Deployments and Updates

*   Write a manifest file named `nginx-exam.yaml` that meets the following requirements:
    *   Starts five replicas that run the `nginx:1.18` image.
    *   Each pod has a label `type=webshop`.
    *   Create a deployment such that while updating, it can temporarily run 8 application instances at the same time, of which 3 should always be available.
    *   The deployment itself should use the label `service=nginx`.
*   Update the deployment to the latest version of the `nginx` image.

### 8. Exposing Applications and Ingress

*   In the `ckad-ns6` namespace, create a deployment that runs the `nginx:1.19` image and give it the name `nginx-deployment`.
*   Ensure it runs 3 replicas.
*   After verifying that the deployment runs successfully, expose it such that users external to the cluster can reach it by addressing the `nodePort` 32000 on the Kubernetes cluster node.
*   Configure Ingress to access the application at `mynginx.info`.

### 9. Network Policies

*   Create a YAML file named `my-nw-policy.yaml` that runs two pods and a network policy.
*   The first pod should run an `nginx` server with default settings.
*   The second pod should run a `busybox` image with the `sleep 3600` command.
*   Use a network policy to restrict traffic between pods in the following way:
    *   Access to the `nginx` server is allowed for the `busybox` pod.
    *   The `busybox` pod itself is not restricted in any way.

### 10. Storage

*   Ensure that everything is created in the `ckad-1311` namespace.
*   Create a PersistentVolume with the name `1311-pv`.
    *   It should provide 2 gigabytes of storage and read-write access to multiple clients simultaneously.
    *   Use the `hostPath` storage type.
*   Create a PersistentVolumeClaim that requests 1 gigabyte from any persistent volume that allows multiple clients simultaneous read-write access.
    *   The name of this object should be `1311-pvc`.
*   Finally, create a pod with the name `1311-pod` that uses this persistent volume.
    *   It should run an `nginx` image and mount the volume on the directory `/web-data`.

### 11. Resource Quotas

*   Create a namespace with the name `limited` in which five pods can be started, and a total amount of 1000 millicores and 2 gigabytes of RAM is available.
*   Run a deployment with the name `restrictginx` in this namespace with three pods, where every pod initially requests 64 megabytes of RAM with an upper limit of 256 megabytes of RAM.

### 12. Canary Deployments

*   Run a deployment with the name `my-web` using the `nginx:1.14` image and 3 replicas.
*   Ensure this deployment is accessible through a service with the name `canary` which uses the `NodePort` service type.
*   Update the deployment to the latest version of `nginx` using the canary deployment update strategy in such a way that 40% of the application offers access to the updated application and 60% still uses the old application.

### 13. Pod Permissions

*   Create a pod manifest file to run a pod with the name `sleepy-box`.
*   It should run the latest version of `busybox` with the `sleep 3600` command as the default command.
*   Ensure that the primary pod user is a member of the supplementary group 2000 while this pod is started.

### 14. Service Accounts

*   Create a pod with the name `allaccess`.
*   Also, create a service account with the name `allaccess`.
*   Ensure that the pod is using the service account.
*   Notice that no further RBAC setup is required.

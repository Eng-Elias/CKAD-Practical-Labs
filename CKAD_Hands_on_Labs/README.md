# CKAD Hands-on Labs: A Practical Guide

This directory contains hands-on lab exercises designed to prepare for the Certified Kubernetes Application Developer (CKAD) exam. The labs are based on the video series available on YouTube.

**Course Playlist**: [CKAD (Certified Kubernetes Application Developer) Hands on LABS](https://www.youtube.com/playlist?list=PLRmSE-IsokP0SQJVyGsNYyUNV3KKwLjrf)

## About the CKAD Exam and These Labs

The CKAD exam consists of approximately 19 labs to be completed in 120 minutes, giving you about 6 minutes per lab on average. To pass, you typically need to solve 12-13 of them correctly. The labs in this course are designed to reflect the real exam's varying difficulty levels:

*   **Easy Labs (1-2 minutes)**: These are fundamental tasks like creating a namespace, a simple pod, or a secret from a literal. You should be able to complete these very quickly.
*   **Medium Labs (3-8 minutes)**: These are the core of the exam. They involve tasks like creating multi-container pods, applying taints and tolerations, or mounting ConfigMaps. You may need to consult the Kubernetes documentation or use `kubectl explain`.
*   **Hard/Long Labs**: These are complex, multi-part questions. While it's good to practice them to build speed and confidence, you don't necessarily need to solve all of them to pass the exam, as long as you perform well on the easier labs.

## Key Topics to Focus On

1.  **ConfigMaps**: A very frequent topic. Be comfortable creating them and mounting them as environment variables or volumes in pods.
2.  **`kubectl run` Command**: Master using `kubectl run` with the `--dry-run=client -o yaml` flags to generate YAML manifests quickly. This is a fundamental skill.
3.  **Pod Commands**: Know how to pass a command to a container at runtime (e.g., `sleep 3600`).
4.  **Volume Mounting**: Practice mounting different types of volumes. The official documentation examples may not always be sufficient, so hands-on practice is key.
5.  **Multi-Container Pods**: Be proficient in editing YAML to define pods with multiple containers, each with its own configuration.
6.  **PersistentVolumes (PV) and PersistentVolumeClaims (PVC)**: Understand how to create and use PVs and PVCs. This is another area where prior practice is more effective than relying on documentation during the exam.
7.  **`kubectl edit`**: Know how and when you can use `kubectl edit` to modify live resources in the cluster. Remember that you cannot edit a running pod's specification directly.

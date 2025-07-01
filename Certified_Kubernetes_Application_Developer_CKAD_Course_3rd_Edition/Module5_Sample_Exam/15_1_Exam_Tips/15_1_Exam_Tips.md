# 15.1 Exam Tips

Here is a summary of the exam tips:

## General Approach

*   **Be Fast:** Prioritize speed over creating "nice" solutions.
    *   Use the most efficient tools available.
    *   Prefer imperative commands (`kubectl create`, `kubectl expose`, etc.) over writing YAML from scratch.
    *   When you need YAML, generate it first and then modify it.

## Using Documentation

*   **`kubernetes.io/docs` is your friend:**
    *   Practice finding information in the official Kubernetes documentation. [    kubernetes.io/docs](https://kubernetes.io/docs/)
    *   You are allowed to copy-paste code snippets from the documentation during the exam.
    *   The exam uses a restricted browser environment, so you can't use your own bookmarks.
    *   Familiarize yourself with the location of complex topics (e.g., mounting a ConfigMap) to save time.
*   **`kubectl` Cheat Sheet:**
    *   The `kubectl` cheat sheet is available in the documentation and is a valuable resource. [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Important `kubectl` Commands

*   **`kubectl explain`:**
    *   Use this command extensively to quickly understand the parameters for different Kubernetes resources.

## Namespaces

*   **Pay Attention to Namespaces:**
    *   Always check the required namespace for each question.
    *   Ensure you are in the correct namespace before creating or modifying resources.
    *   Resources created in the wrong namespace will not be graded correctly.

## Exam Environment

*   **Allowed Websites:**
    *   Stick to the list of allowed websites provided at the beginning of the exam.
    *   Navigating to unauthorized sites (e.g., `discuss.kubernetes.io`) can lead to warnings or disqualification.
    *   `kubernetes.io/docs` should be your primary resource.

# How to Debug Kubernetes Deployment (Step-by-Step Guide)

Deploying apps to Kubernetes is exciting—but sometimes, it feels like hitting a wall when things go wrong. You type `kubectl apply`, and… the pod just won't run. Frustrating, right?

Whether you’re a beginner or someone who has already worked with Kubernetes, debugging deployment errors can feel like navigating a maze. But don’t worry. This guide will **walk you through how to debug Kubernetes deployment**, step by step, using **simple commands**, real examples, and **no complicated jargon**.

---

## Why Debugging Kubernetes Deployment Matters

Kubernetes is powerful. But like any complex system, it's sensitive to small errors—wrong indentations, misnamed images, or missing probes. A small mistake in your YAML file can crash the entire pod.

**Here’s the truth:** If you don’t learn how to debug your deployments, you’ll waste hours trying to guess what went wrong. That’s why this guide exists—to save your time and sanity.

---

## Common Kubernetes Deployment Issues

Before you fix something, you need to know what to look for. Here are some **common errors in Kubernetes deployments**:

* **ImagePullBackOff**: Kubernetes can’t pull the container image.
* **CrashLoopBackOff**: The container starts and crashes again and again.
* **Pending Pods**: Pod is stuck waiting to be scheduled.
* **Misconfigured YAML**: One space or wrong key can break the deployment.
* **Service Not Reachable**: Pods are running, but you can’t access the service.

---

## 🛠 Step-by-Step Guide to Debug Kubernetes Deployment

Let’s go through each debugging step like a checklist.

---

### 1. **Check Pod Status First**

Use this command:

```bash
kubectl get pods
```

You’ll see something like this:

```
my-app-65d7bcd75b-x9k8p   CrashLoopBackOff
```

This status tells you if the pod is crashing, waiting, or running. Look for keywords like **Error**, **Pending**, or **CrashLoopBackOff**.

---

### 2. **Use Describe to Find the Root Cause**

Run this:

```bash
kubectl describe pod <pod-name>
```

This shows details like:

* Events at the bottom (very useful!)
* Node assignment
* Reason for failure

Look for lines like:

```
Failed to pull image "myapp:v1": image not found
```

Now you know your image name might be wrong or the repo is private.

---

### 3. **Check Container Logs**

If the pod is running (even briefly), logs help a lot.

```bash
kubectl logs <pod-name>
```

If your pod keeps crashing, use:

```bash
kubectl logs <pod-name> --previous
```

**Tip**: If your pod has multiple containers, use:

```bash
kubectl logs <pod-name> -c <container-name>
```

Logs show real-time messages from your app. It could be as simple as a missing config or failed database connection.

---

### 4. **Review Deployment Status**

Check rollout progress:

```bash
kubectl rollout status deployment <deployment-name>
```

If rollout failed:

```bash
kubectl describe deployment <deployment-name>
```

You’ll see messages like:

```
Minimum number of live pods was not available
```

This means not enough pods were healthy during the rollout.

---

### 5. **Check Kubernetes Events**

This one is gold.

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

This will list all recent events in order, like:

* Pod failed to schedule
* ImagePull errors
* Node resource limits

**It’s like reading the diary of your cluster.**

---

### 6. **Validate YAML Configuration**

YAML is strict. One bad indent, and nothing works.

Use this:

```bash
kubectl apply --dry-run=client -f deployment.yaml
```

This checks your YAML without actually deploying it.

Also, consider using a YAML linter or VS Code extension to catch common mistakes.

---

### 7. **Check Resource Limits and Quotas**

Pods may fail if your node doesn’t have enough memory or CPU.

Use:

```bash
kubectl describe quota
```

Also check your pod’s spec:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

If these values are too high, the pod may not be scheduled.

---

### 8. **Verify Networking and Services**

Check if services are exposing your pods correctly.

```bash
kubectl get svc
kubectl get ep
```

You can also exec into the pod and use curl or ping:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

Then inside:

```bash
curl http://<service-name>:<port>
```

---

### 9. **Check Readiness and Liveness Probes**

If probes fail, Kubernetes restarts the pod.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

Make sure your app actually responds to `/healthz` or whatever path you set. Otherwise, your pod will constantly crash even if it’s working fine.

---

## Advanced Kubernetes Debugging Techniques

Want to go deeper? Here are a few powerful tools and tricks:

* **`kubectl debug`**: Add an ephemeral container to inspect pods
* **Lens or K9s**: Visual dashboards for easier navigation
* **`stern`**: Tail logs of multiple pods in real-time
* **Prometheus & Grafana**: See metrics like CPU, memory, restarts
* **Fluentd or Loki**: Centralized log aggregation

---

## Tools That Make Debugging Easier

| Tool            | Use Case                        |
| --------------- | ------------------------------- |
| `kubectl`       | Default CLI for most debugging  |
| `kubectl-debug` | Attach debug container          |
| K9s             | Terminal UI to browse resources |
| Stern           | Live tailing of pod logs        |
| VS Code         | Built-in Kubernetes integration |

---

## Real-World Example

Let’s say your pod has this error:

```
ImagePullBackOff
```

Here’s how you solve it:

1. Run `kubectl describe pod`
2. Read the event log:

```
Failed to pull image "app:latest": repository does not exist
```

3. Fix your YAML:

```yaml
image: myrepo/app:1.0.0
```

4. Apply again:

```bash
kubectl apply -f deployment.yaml
```

Problem solved in less than 5 minutes.

---

## Best Practices to Avoid Deployment Failures

* Always double-check your YAML syntax
* Use CI/CD to test deployments before pushing
* Set proper health probes
* Monitor resource usage
* Use labels and annotations wisely
* Use versioned image tags (not “latest”)

---

## Conclusion: Debugging = Skill + Curiosity

Debugging a Kubernetes deployment doesn’t need to feel scary or confusing. Yes, the error messages can be overwhelming at first. But with a clear process and the right commands, you’ll learn to **see patterns**, **solve issues quickly**, and grow as a DevOps engineer.

**The key takeaway:** Next time your deployment breaks, don’t panic. Open your terminal, breathe, and follow the steps above. You got this.

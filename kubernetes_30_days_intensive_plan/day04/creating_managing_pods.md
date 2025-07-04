# Creating and Managing Pods in Kubernetes

## What is a Pod?
A **Pod** is the smallest and simplest unit in the Kubernetes object model that you can create or deploy. A Pod represents a single instance of a running process in your cluster. It can contain one or more tightly coupled containers that share the same network namespace and storage.

---

## Why Pods?
- To host single or multiple containers that must run together.
- Shared storage (Volumes) and network for tightly coupled apps.
- Acts as a wrapper around containers to manage them in Kubernetes.

---

## Creating Pods
There are multiple ways to create pods:

### 1. Using `kubectl run`
```bash
kubectl run myapp --image=nginx --restart=Never
```
> This creates a single pod named `myapp` with the `nginx` container.

### 2. Using a YAML Manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: nginx
      ports:
        - containerPort: 80
```
Apply the file:
```bash
kubectl apply -f pod.yaml
```

---

## Verifying Pods
```bash
kubectl get pods
kubectl describe pod myapp-pod
kubectl logs myapp-pod
```

---

## Managing Pods

### Deleting a Pod
```bash
kubectl delete pod myapp-pod
```

### Executing Commands in a Pod
```bash
kubectl exec -it myapp-pod -- /bin/bash
```

### Viewing Logs
```bash
kubectl logs myapp-pod
```

### Restarting a Pod
Pods are not designed to be restarted manually. Instead, delete the pod and it will be recreated by a higher-level controller (like Deployment) if managed.

```bash
kubectl delete pod myapp-pod
```

---

## Best Practices
- **Use Deployments** to manage Pods in production.
- Use **labels and selectors** to manage groups of pods.
- Monitor Pod status using `kubectl get pods`.
- Avoid creating standalone pods unless absolutely necessary.

---

## Example: Multi-Container Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: app-container
      image: nginx
    - name: sidecar-container
      image: busybox
      command: ['sh', '-c', 'while true; do echo hello; sleep 10; done']
```

---

## Summary
- A Pod is a group of one or more containers.
- Use `kubectl run` for simple cases, or YAML for production-ready specs.
- Pods are ephemeral; use controllers like Deployments for lifecycle management.

---

## Useful Commands
| Action | Command |
|--------|---------|
| Create Pod | `kubectl apply -f pod.yaml` |
| List Pods | `kubectl get pods` |
| Delete Pod | `kubectl delete pod <name>` |
| Pod Logs | `kubectl logs <pod-name>` |
| Exec into Pod | `kubectl exec -it <pod-name> -- /bin/bash` |


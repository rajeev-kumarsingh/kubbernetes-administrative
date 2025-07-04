# Replication Controllers, ReplicaSets, and Deployments in Kubernetes

## 1️⃣ Replication Controllers

### What it is:

ReplicationController (RC) ensures that a specified number of pod replicas are running at any given time.

**Features:**

- Ensures pod availability.
- Automatically replaces failed or deleted pods.
- Supports scaling by modifying `replicas`.
- **Older Kubernetes resource** (now replaced by ReplicaSets and Deployments).

### Example:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Commands:

```bash
kubectl apply -f nginx-rc.yaml
kubectl get rc
kubectl delete rc nginx-rc
```

---

## 2️⃣ ReplicaSets

### What it is:

ReplicaSet (RS) is the **next-generation controller replacing ReplicationController**.

**Features:**

- Ensures a specified number of replicas are running.
- Supports **set-based selectors**.
- Commonly managed by Deployments for easier updates.

### Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Commands:

```bash
kubectl apply -f nginx-rs.yaml
kubectl get rs
kubectl describe rs nginx-rs
kubectl delete rs nginx-rs
```

---

## 3️⃣ Deployments

### What it is:

Deployment provides **declarative updates for Pods and ReplicaSets**.

**Features:**

- Manages ReplicaSets internally.
- Supports rolling updates and rollbacks.
- Easy scaling and updates without downtime.
- Allows tracking and history of deployments.

### Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Commands:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl rollout status deployment nginx-deployment
kubectl rollout history deployment nginx-deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl delete deployment nginx-deployment
```

---

## Summary Table

| Feature                  | ReplicationController  | ReplicaSet                    | Deployment                    |
| ------------------------ | ---------------------- | ----------------------------- | ----------------------------- |
| Ensures replica count    | ✅                      | ✅                             | ✅                             |
| Selector types           | Equality-based         | Set-based                     | Set-based                     |
| Supports rolling updates | ❌                      | ❌                             | ✅                             |
| Rollback support         | ❌                      | ❌                             | ✅                             |
| Replacement strategy     | Older, rarely used now | Usually managed by Deployment | Recommended for managing pods |

---

## Practical Lab:

✅ Create a `nginx-deployment.yaml`, apply it, scale it, update the image, and test rollback:

```bash
# Apply deployment
touch nginx-deployment.yaml # paste the above deployment YAML
kubectl apply -f nginx-deployment.yaml

# Scale to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# Update image
kubectl set image deployment nginx-deployment nginx=nginx:1.26 --record

# Check rollout status
kubectl rollout status deployment nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment nginx-deployment
```

---

## Key Takeaways

✅ **Always use Deployments in production** for safe updates and easy rollback. ✅ ReplicaSets replace RCs and are typically not used directly. ✅ Understanding these controllers is key for Kubernetes exam readiness and real-world workload management.

---

If you would like, I can prepare a **hands-on AWS or Minikube lab exercise guide with these YAMLs to practice end-to-end** for your Kubernetes learning.


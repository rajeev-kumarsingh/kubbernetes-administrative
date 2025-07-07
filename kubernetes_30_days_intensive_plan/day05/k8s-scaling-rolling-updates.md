
# Kubernetes: Scaling and Rolling Updates

## 📈 1. Scaling in Kubernetes

**Scaling** refers to increasing or decreasing the number of pod replicas in a Deployment or ReplicaSet.

### 🔧 Manual Scaling Example

```bash
# Scale a deployment to 5 replicas
kubectl scale deployment my-app --replicas=5
```

### 🔁 Example Deployment YAML for Scaling

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx
        ports:
        - containerPort: 80
```

You can increase the number of replicas in the above file and apply using:

```bash
kubectl apply -f deployment.yaml
```

## ⚙️ 2. Rolling Updates in Kubernetes

**Rolling Updates** allow updating pods incrementally with zero downtime by gradually replacing older versions with new ones.

### 🔁 Default Rolling Update Strategy

Kubernetes uses rolling update strategy by default when you run:

```bash
kubectl set image deployment/my-app my-app-container=nginx:1.21
```

This replaces the image gradually while maintaining availability.

### ⚙️ Rolling Update Configuration in YAML

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

- `maxSurge`: Extra pods that can be created during update (default: 25%)
- `maxUnavailable`: Number of pods that can be unavailable during update (default: 25%)

### 🔄 Check Update Progress

```bash
kubectl rollout status deployment/my-app
```

### ⏪ Rollback if Needed

```bash
kubectl rollout undo deployment/my-app
```

---

## ✅ Summary

| Feature        | Command Example                             | Purpose                                |
|----------------|---------------------------------------------|----------------------------------------|
| Scale Up       | `kubectl scale deployment my-app --replicas=5` | Increase number of pod replicas        |
| Update Image   | `kubectl set image deployment/my-app ...`   | Rolling update with zero downtime      |
| Monitor Update | `kubectl rollout status deployment/my-app`  | Track the rollout progress             |
| Rollback       | `kubectl rollout undo deployment/my-app`    | Revert to previous stable state        |


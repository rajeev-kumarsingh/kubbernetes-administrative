# 🐤 Canary Deployment: Detailed Explanation with Real-World Example

## What is Canary Deployment?

**Canary Deployment** is a **progressive delivery** technique used in software releases. Instead of deploying a new version of your application to all users at once, you gradually roll it out to a small subset of users first (the "canaries"). If there are no issues, you roll it out to more users until it's fully deployed.

---

## 🎯 Why Use Canary Deployment?

- Reduce risk of outages or bugs in production.
- Get early feedback from real users.
- Easy rollback if issues are detected.

---

## 🔧 How It Works (Conceptual Flow)

1. Current version (v1) is running in production.
2. Deploy new version (v2) to a small percentage of users.
3. Monitor performance, errors, logs, etc.
4. If everything looks good, slowly increase traffic to v2.
5. Eventually deprecate v1 and make v2 the default.

---

## 📦 Real-World Example Using Kubernetes + NGINX Ingress

Let’s say we have a microservice called `web-app`. Here's how you would perform a canary deployment using Kubernetes:

### ✅ Step-by-step Setup

### 1. Current Live Version (`v1`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
      version: v1
  template:
    metadata:
      labels:
        app: web-app
        version: v1
    spec:
      containers:
      - name: app
        image: myrepo/web-app:v1
        ports:
        - containerPort: 80
```

### 2. Canary Version (`v2`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
      version: v2
  template:
    metadata:
      labels:
        app: web-app
        version: v2
    spec:
      containers:
      - name: app
        image: myrepo/web-app:v2
        ports:
        - containerPort: 80
```

### 3. Service to Load Balance Between v1 and v2

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 4. Ingress with Canary Annotation (for NGINX Ingress)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service
            port:
              number: 80
```

> You would use **two Ingress resources**:
> - One for regular traffic (v1)
> - One with `canary: true` for directing a small percentage to v2.

---

## 🧪 Monitoring Tools

- **Prometheus + Grafana** for metrics
- **Elastic Stack** or **Datadog** for logs
- **Sentry** or **New Relic** for error monitoring

---

## 🔄 Gradual Rollout

- Increase `canary-weight` to 25%, then 50%, then 100%.
- Eventually, retire `v1` deployment.

If issues are found:

- Set `canary-weight: 0` or delete `v2` deployment.

---

## 🔙 Rollback Strategy

- Remove the `v2` Deployment or set its replicas to 0.
- Update ingress to remove canary annotations.

---

## 🧵 Summary

| Step            | Action                                                |
|-----------------|-------------------------------------------------------|
| 1. Deploy v2    | Create new version with small replica count           |
| 2. Update Ingress | Add canary annotations with small traffic weight   |
| 3. Monitor      | Use observability tools for health and error tracking |
| 4. Rollout/Rollback | Increase weight or disable canary based on results |
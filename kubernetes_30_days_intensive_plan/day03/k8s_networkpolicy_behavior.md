
# 🛡️ Kubernetes NetworkPolicy Behavior

This document explains what happens when you **do or do not apply a NetworkPolicy** in a Kubernetes cluster, using a frontend-backend example.

---

## 📌 Default Behavior (No NetworkPolicy)

By default, Kubernetes allows **all Pods to communicate with each other**.

| Source Pod     | Destination Pod | Access Allowed? |
|----------------|------------------|------------------|
| `frontend`     | `backend` (via Service or Pod IP) | ✅ Yes |
| Any Pod        | Any other Pod    | ✅ Yes |

### ✅ Example

- A `frontend` Pod can freely communicate with a `backend` Pod using its Service:
```bash
kubectl exec -it frontend -- curl backend-service
```
- This will return a response (e.g., default Nginx page if backend is Nginx).

---

## 🔐 With NetworkPolicy Applied

Once you apply a `NetworkPolicy`, **Kubernetes defaults to "deny all"** for Pods that are selected by the policy.

Only the traffic **explicitly allowed** in the policy is permitted.

### 🧱 Example NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

### 🔍 What this does:

- Allows only Pods with `app=frontend` label to access Pods with `app=backend`.
- All other traffic is **blocked**.

### 🔄 Behavior After Applying Policy

| Source Pod     | Destination Pod | Access Allowed? |
|----------------|------------------|------------------|
| `frontend`     | `backend`        | ✅ Yes |
| `debug` pod    | `backend`        | ❌ No |
| Any other Pod  | `backend`        | ❌ No |

---

## ✅ Summary

| Scenario                    | Communication Allowed? |
|-----------------------------|-------------------------|
| No NetworkPolicy            | ✅ All Pods can communicate |
| With NetworkPolicy applied  | ❌ Only allowed traffic is permitted |

---

## 🔧 Useful Commands

```bash
kubectl get networkpolicy
kubectl describe networkpolicy <name>
kubectl run curl --image=radial/busyboxplus:curl -it --restart=Never -- sh
curl backend-service
```

---

## 📌 Best Practices

- Start with deny-all, then allow specific traffic.
- Apply NetworkPolicies per namespace.
- Always label Pods properly for use in policies.

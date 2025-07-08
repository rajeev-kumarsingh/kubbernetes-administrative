
# 📘 Kubernetes Services and Networking Explained

---

## 🧩 What is a Service in Kubernetes?

A **Service** in Kubernetes is an abstraction that defines a logical set of Pods and a policy by which to access them. Services enable communication between different components in your application, or between external users and the application.

---

## 🚦 Why Do We Need Services?

Pods are **ephemeral** and their IP addresses can change. So, to access a Pod reliably (even after rescheduling or restarting), Kubernetes provides **Services**.

---

## 🧰 Types of Kubernetes Services

---

### 1. **ClusterIP (Default)**

- **Access**: Internal-only (within the cluster)
- **Use case**: Communication between pods (e.g., frontend -> backend)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

➡️ This creates an internal IP address that can be used by other pods in the cluster.

---

### 2. **NodePort**

- **Access**: Exposes the service on each Node’s IP at a static port.
- **Use case**: Basic external access (via `<NodeIP>:<NodePort>`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
  type: NodePort
```

➡️ Access it with: `http://<NodeIP>:30080`

---

### 3. **LoadBalancer**

- **Access**: Provisions an external load balancer from cloud provider (e.g., AWS ELB).
- **Use case**: Production-grade public-facing apps.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-lb
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

➡️ Automatically gets an **external IP** (from cloud provider).

---

### 4. **ExternalName**

- **Access**: Maps service to an external DNS name.
- **Use case**: Redirect traffic to an external database or API.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db
spec:
  type: ExternalName
  externalName: db.externalhost.com
```

➡️ DNS for `my-db.default.svc.cluster.local` resolves to `db.externalhost.com`.

---

## 🌐 Kubernetes Networking Basics

---

### 🔁 Pod-to-Pod Communication

- Every Pod gets its own IP.
- Can communicate **without NAT** (flat network).
- Uses **CNI (Container Network Interface)** plugin (e.g., Calico, Flannel).

---

### 🧭 DNS in Kubernetes

- Every service and pod gets a DNS name.
- DNS format: `service-name.namespace.svc.cluster.local`

🔍 Example:

```
frontend calls backend: http://backend-service.default.svc.cluster.local
```

---

### 🧱 NetworkPolicies

Define **rules** for traffic flow (like firewalls).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
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

➡️ Only allows `frontend` to talk to `backend`.

---

## 🛠️ Real-World Example

Let’s say you have a frontend and backend app. The frontend needs to access the backend.

- Deploy Pods with labels `app: frontend` and `app: backend`
- Create `backend-service` (ClusterIP)
- Frontend calls backend via `http://backend-service`

---

### 🗂️ Summary Table

| Service Type   | Internal | External | Use Case                          |
|----------------|----------|----------|-----------------------------------|
| ClusterIP      | ✅       | ❌       | Internal app communication        |
| NodePort       | ✅       | ✅       | Simple external access            |
| LoadBalancer   | ✅       | ✅       | Cloud-native public access        |
| ExternalName   | ❌       | ✅       | Linking to external services      |

---

## 📎 Download Markdown File

[Click here to download `k8s-services-networking.md`](sandbox:/mnt/data/k8s-services-networking.md)

---

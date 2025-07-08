
# 🔍 Kubernetes Service YAML Explanation

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

---

## 📘 Explanation

### 🔹 apiVersion: v1
Specifies the Kubernetes API version. For Service objects, `v1` is standard.

### 🔹 kind: Service
This tells Kubernetes you are defining a **Service** object.

### 🔹 metadata.name: my-service
Name of the service object. It will be accessible within the cluster using this name via DNS:
```
my-service.namespace.svc.cluster.local
```

### 🔹 spec.selector
```yaml
selector:
  app.kubernetes.io/name: MyApp
```
This is how the Service finds the target Pods. It looks for Pods with the label:
```
app.kubernetes.io/name=MyApp
```

### 🔹 spec.ports
```yaml
ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

| Field         | Value  | Description                                                    |
|---------------|--------|----------------------------------------------------------------|
| protocol      | TCP    | Protocol used by the service. Default is TCP.                 |
| port          | 80     | The port exposed by the Service inside the cluster.           |
| targetPort    | 9376   | The port on the Pod where traffic is forwarded.               |

➡️ Traffic sent to `my-service:80` is forwarded to `9376` on matched Pods.

### 🔹 type (Implicit)
Since `type` is not specified, it defaults to `ClusterIP`. This means:

- The Service is **internal-only** to the cluster.
- External users cannot access it directly.

---

## ✅ Summary

This Service:

- Matches Pods with label `app.kubernetes.io/name=MyApp`
- Listens on port `80`
- Forwards traffic to container port `9376`
- Is only accessible **within the cluster** (ClusterIP type)

---

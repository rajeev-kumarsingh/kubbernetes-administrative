
# 📘 Kubernetes Service Ports Explained

In Kubernetes, Services are used to expose applications running on Pods. There are three main port-related fields in a Service spec:

- `port`
- `targetPort`
- `nodePort` (used only in `NodePort` type services)

---

## 🔧 Field Breakdown

### 1. `port`
- This is the **port that the Service exposes** inside the cluster.
- It is the port that other applications in the cluster will use to access the Service.

```yaml
port: 80
```
> This means the service is listening on port 80.

---

### 2. `targetPort`
- This is the **port on the Pod** to which the traffic is forwarded.
- It tells the Service where to send the traffic **inside the Pod**.
- It can be a number or a name (if named port is defined in the container spec).

```yaml
targetPort: 8080
```
> This means traffic from Service port 80 is forwarded to container's port 8080.

---

### 3. `nodePort` (Only for `NodePort` type Services)
- This is the port on **each Node** where the Service can be accessed externally.
- It must be between **30000 and 32767** unless configured otherwise.
- If not specified, Kubernetes automatically assigns one in that range.

```yaml
nodePort: 31234
```
> This exposes the service on port `31234` on **every Node IP**.

---

## 🧪 Example: NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80          # Service port (cluster internal)
      targetPort: 8080  # Container port
      nodePort: 31234   # External Node IP port
```

- Clients inside the cluster use: `my-nodeport-service:80`
- External clients use: `<NodeIP>:31234`

---

## 🔁 Summary Table

| Field       | Description                             | Example |
|-------------|-----------------------------------------|---------|
| `port`      | Port exposed by the Service             | 80      |
| `targetPort`| Pod's container port                    | 8080    |
| `nodePort`  | Port on the Node (external access)      | 31234   |

---

## 🧠 Bonus: ClusterIP Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

- Accessible **only within the cluster** as `my-clusterip-service:80`.

---

## ✅ Summary
- `port`: Service port inside the cluster.
- `targetPort`: Container’s actual port inside the pod.
- `nodePort`: Exposes the service externally on a port of the node (30000–32767).

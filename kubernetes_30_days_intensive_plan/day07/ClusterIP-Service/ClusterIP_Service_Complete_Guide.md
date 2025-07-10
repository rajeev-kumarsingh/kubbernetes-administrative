# Kubernetes ClusterIP Service - Complete Guide

A **ClusterIP** service is the default type of Kubernetes service. It provides a way to expose an application running in a set of Pods **internally** within the Kubernetes cluster. This means it **cannot be accessed directly from outside the cluster**.

---

## 📌 Key Characteristics of ClusterIP

- **Internal Access Only**: Accessible only within the Kubernetes cluster. Not exposed externally.
- **Default Service Type**: If you don’t specify `type` in a service definition, it becomes `ClusterIP` by default.
- **Stable Virtual IP (VIP)**: Kubernetes assigns a virtual IP address to the service, acting as a single entry point.
- **Load Balancing**: Evenly distributes traffic among the backend Pods (via label selectors).
- **No Need to Know Pod IPs**: Clients use the service name or IP; they don’t need to know each Pod’s IP address.

---

## 🔧 YAML Definition Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80        # Port exposed by the service
      targetPort: 8080 # Port on the Pod
  type: ClusterIP
```

> If `type` is omitted, ClusterIP is assumed.

---

## 🧠 How It Works Internally

1. **Label Selector**: The `selector` field maps the service to the matching Pods.
2. **Endpoints Object**: Kubernetes creates an Endpoints object with the IPs of matched Pods.
3. **Virtual IP**: Kubernetes assigns a ClusterIP to the service.
4. **kube-proxy**: It uses iptables or IPVS rules to forward requests to one of the Pods behind the service.

---

## 🌐 Accessing a ClusterIP Service

- Accessible only **within the cluster**.
- You can access it using:
  - The service name (e.g., `my-clusterip-service`) from other Pods in the same namespace.
  - FQDN: `my-clusterip-service.my-namespace.svc.cluster.local`

---

## 🧪 Use Case Scenarios

- Internal microservice communication
- Backend services (e.g., databases, internal APIs)
- DNS resolution within the cluster

---

## 🛠️ Troubleshooting Tips

| Issue                             | Possible Cause                              | Solution                                   |
|----------------------------------|---------------------------------------------|--------------------------------------------|
| Cannot connect to service        | Pod selector mismatch                       | Check the labels on your Pods              |
| Service returns no endpoints     | No matching Pods                            | Verify that Pods are running and labeled   |
| Connection refused               | Port mismatch                               | Ensure `port` and `targetPort` are correct |
| DNS resolution fails             | CoreDNS issue or misconfiguration           | Check CoreDNS logs and config              |

---

## 🔍 How to View ClusterIP Services

```bash
kubectl get svc
kubectl describe svc <service-name>
```

---

## 🧹 Clean-Up

To delete a ClusterIP service:

```bash
kubectl delete svc my-clusterip-service
```

---

## ✅ Summary

| Feature             | ClusterIP                       |
|---------------------|----------------------------------|
| Internal Access     | ✅ Yes                          |
| External Access     | ❌ No                           |
| Default Type        | ✅ Yes                          |
| Load Balancing      | ✅ Yes (across matched Pods)   |
| DNS Support         | ✅ Yes                          |

ClusterIP is the most secure and common way to expose services **within** your cluster. It forms the foundation of Kubernetes service discovery and communication.
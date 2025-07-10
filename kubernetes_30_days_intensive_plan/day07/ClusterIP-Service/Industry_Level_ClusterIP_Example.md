# 🏢 Industry-Level Example: ClusterIP Service in Kubernetes

## 🚀 Use Case: E-commerce Microservices Architecture

This example demonstrates how a real-world e-commerce application uses Kubernetes **ClusterIP services** for secure internal communication between microservices.

---

## 🧩 Architecture Overview

### Microservices:

- `frontend`: UI for customers
- `product-catalog`: Manages product data
- `user-service`: Manages user data
- `order-service`: Manages orders
- `mysql`: Stores data for `product-catalog` (and others)

`product-catalog` connects to `mysql` **internally** using a **ClusterIP** service.

---

## 🔧 MySQL Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: rootpassword
          ports:
            - containerPort: 3306
```

---

## 🔌 MySQL ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

> ✅ This makes MySQL accessible **only** within the cluster — no external exposure.

---

## 📦 Product-Catalog Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      containers:
        - name: product-catalog
          image: myorg/product-catalog:1.0
          env:
            - name: DB_HOST
              value: mysql # Refers to the ClusterIP service
            - name: DB_USER
              value: root
            - name: DB_PASSWORD
              value: rootpassword
          ports:
            - containerPort: 3000
```

---

## 🌐 Internal Communication Code

Example (Node.js):

```js
const mysql = require("mysql2");

const connection = mysql.createConnection({
  host: "mysql", // Service name = ClusterIP hostname
  user: "root",
  password: "rootpassword",
  database: "products",
});

connection.connect((err) => {
  if (err) throw err;
  console.log("Connected to MySQL inside Kubernetes!");
});
```

---

## ✅ Why Use ClusterIP Here?

| Feature          | Benefit                                             |
| ---------------- | --------------------------------------------------- |
| 🔒 Secure        | Database is not exposed to internet                 |
| 🌐 DNS-based     | Services resolve using names like `mysql`           |
| ♻️ Stable IP     | ClusterIP stays constant even if Pod IPs change     |
| 🎯 Load Balanced | Multiple Pod replicas can be targeted automatically |

---

## 🛠️ Useful Commands

```bash
kubectl get svc
kubectl describe svc mysql
kubectl get pods -l app=mysql
```

---

## 🧹 Clean Up Resources

```bash
kubectl delete deployment mysql
kubectl delete service mysql
kubectl delete deployment product-catalog
```

---

## 📌 Summary

- **ClusterIP** services are perfect for **internal service-to-service communication**.
- Common in production setups for databases, APIs, caches, etc.
- Secure, stable, and DNS-resolved.

---

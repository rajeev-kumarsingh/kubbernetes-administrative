# 🏢 Industry-Level Example: ClusterIP Service in Kubernetes

## 🚀 Use Case: E-commerce Microservices Architecture

This example demonstrates how a real-world e-commerce application uses Kubernetes **ClusterIP services** for secure internal communication between microservices, now enhanced with:

- ✅ PersistentVolumeClaims (PVCs)
- ✅ ConfigMaps for environment config
- ✅ Ingress for frontend exposure

---

## 🧩 Architecture Overview

### Microservices:
- `frontend`: UI for customers (exposed via Ingress)
- `product-catalog`: Manages product data
- `mysql`: Stores product data for `product-catalog`

`product-catalog` connects to `mysql` using a **ClusterIP** service. The frontend accesses `product-catalog` via internal communication.

---

## 🔐 ConfigMap for Product Catalog

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: product-config
data:
  DB_HOST: mysql
  DB_USER: root
  DB_NAME: products
```

---

## 🔐 Secret for Database Password

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cm9vdHBhc3N3b3Jk  # base64 for 'rootpassword'
```

---

## 💾 Persistent Volume Claim for MySQL

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 🔧 MySQL Deployment (with PVC)

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
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: mysql-storage
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```

---

## 🔌 MySQL ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

---

## 📦 Product Catalog Deployment (with ConfigMap & Secret)

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
          valueFrom:
            configMapKeyRef:
              name: product-config
              key: DB_HOST
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: product-config
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
        - name: DB_NAME
          valueFrom:
            configMapKeyRef:
              name: product-config
              key: DB_NAME
        ports:
        - containerPort: 3000
```

---

## 🔐 Product Catalog ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: product-catalog
spec:
  selector:
    app: product-catalog
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

---

## 🌐 Frontend Deployment (Exposed via Ingress)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myorg/frontend:1.0
        ports:
        - containerPort: 80
```

---

## 🌐 Frontend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

## 🌐 Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: ecommerce.local
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend
              port:
                number: 80
```

> ⚠️ You must have an Ingress Controller (e.g., nginx-ingress) installed and DNS configured or use `/etc/hosts` for `ecommerce.local`.

---

## ✅ Summary

| Component      | Description                                      |
|----------------|--------------------------------------------------|
| **PVC**        | Ensures persistent storage for MySQL             |
| **ConfigMap**  | Injects non-sensitive configuration              |
| **Secret**     | Injects DB credentials securely                  |
| **Ingress**    | Exposes `frontend` publicly via domain routing   |
| **ClusterIP**  | Used for internal service discovery & networking |

---

Let me know if you want this turned into a `.zip` with separate YAML files too!
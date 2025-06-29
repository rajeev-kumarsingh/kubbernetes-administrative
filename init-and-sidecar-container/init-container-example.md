
# 🚀 Real-World Example: Kubernetes Init Container

This document demonstrates a **real-world example** of using an **Init Container** in Kubernetes to initialize a MySQL database **before** starting the main application container.

---

## 🔸 Use Case

You have a web application (`my-app`) that connects to a MySQL database. You want to initialize the database with a schema or seed data when the pod starts — but only before the main app container runs.

---

## 🧱 Architecture Overview

```text
┌─────────────────────────────┐
│      Kubernetes Pod         │
│ ┌─────────────────────────┐ │
│ │     Init Container      │ │
│ │   (runs SQL script)     │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │     App Container       │ │
│ │ (waits for init to end) │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

---

## 📝 Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      initContainers:
      - name: init-database
        image: mysql:5.7
        command: ['sh', '-c', 'mysql -h db -u root -p$MYSQL_ROOT_PASSWORD < /scripts/init.sql']
        volumeMounts:
        - name: init-scripts
          mountPath: /scripts
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: root-password
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: init-scripts
          mountPath: /scripts
      volumes:
      - name: init-scripts
        configMap:
          name: init-scripts
```

---

## 📦 Required Kubernetes Resources

### 1. ConfigMap (for SQL script)

```bash
kubectl create configmap init-scripts --from-file=init.sql=./init.sql
```

Or as YAML:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: init-scripts
data:
  init.sql: |
    CREATE DATABASE IF NOT EXISTS myapp;
    USE myapp;
    CREATE TABLE users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(255)
    );
```

### 2. Secret (for MySQL root password)

```bash
kubectl create secret generic mysql-secrets --from-literal=root-password=YourStrongPassword
```

---

## ⏱ Execution Order

1. **Init Container (`init-database`)** runs first.
2. Executes `init.sql` against the MySQL service at host `db`.
3. **Main Container (`my-app`)** starts **only if** the init container exits successfully.

---

## ✅ Benefits of Init Containers

- Isolate init logic from main app logic.
- Ensure database is ready or initialized before the app runs.
- Reuse Docker images (like `mysql`) for setup logic.

---

## 📌 Tips

- Init containers **always run to completion** before the main containers start.
- If an init container fails, the pod is restarted based on the `restartPolicy`.

---

## 🔚 Conclusion

Init containers are a powerful Kubernetes feature to prepare runtime environments. They are ideal for tasks like:

- DB initialization
- File permission setup
- Waiting for services to become available

Use them to build robust, reliable, and self-initializing pods.

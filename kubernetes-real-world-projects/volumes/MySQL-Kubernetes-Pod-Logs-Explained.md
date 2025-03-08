# MySQL Kubernetes Pod Logs Explained

## **1. MySQL Entrypoint Script Starts**

```log
2025-03-08 17:11:58+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.41-1.el9 started.
```

- MySQL Docker container's entrypoint script starts, initializing configurations.

```log
2025-03-08 17:11:58+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
```

- Container switches to **mysql** user for security.

---

## **2. MySQL Initialization Begins**

```log
2025-03-08 17:11:58+00:00 [Note] [Entrypoint]: Initializing database files
```

- MySQL initializes the database files as it's the first run.

```log
2025-03-08T17:11:58.704566Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.41) initializing of server in progress...
```

- MySQL database initialization begins.

```log
2025-03-08T17:11:59.954977Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password!
```

- Root user is created without a password (should be changed for security).

```log
2025-03-08 17:12:02+00:00 [Note] [Entrypoint]: Database files initialized
```

- MySQL database files are successfully initialized.

---

## **3. MySQL Temporary Server Starts**

```log
2025-03-08 17:12:02+00:00 [Note] [Entrypoint]: Starting temporary server
```

- A **temporary MySQL server** is started to finalize setup.

```log
2025-03-08T17:12:02.608830Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.41) starting as process 128
```

- Temporary MySQL server is launched.

```log
2025-03-08T17:12:03.095590Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
```

- Warning: **Self-signed CA certificate** detected (should be replaced with a trusted one).

---

## **4. Time Zone Data Warning**

```log
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
```

- MySQL attempts to load **time zone data**, but some files are missing.

---

## **5. Temporary Server Stops**

```log
2025-03-08 17:12:04+00:00 [Note] [Entrypoint]: Stopping temporary server
```

- Temporary MySQL server stops after initialization.

```log
2025-03-08T17:12:05.766487Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete.
```

- Temporary server shutdown is successful.

---

## **6. MySQL Final Startup**

```log
2025-03-08 17:12:06+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.
```

- MySQL is now **ready to start** normally.

```log
2025-03-08T17:12:06.554683Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.41' socket: '/var/run/mysqld/mysqld.sock' port: 3306.
```

- **MySQL is fully up and running** on port **3306**, ready for connections.

---

## **Next Steps**

### **Check MySQL Pod Status**

```sh
kubectl get pods
```

### **Access MySQL inside the Pod**

```sh
kubectl exec -it mysql-pod -- mysql -uroot -p
```

### **Fix Time Zone Files (If Needed)**

```sh
kubectl exec -it mysql-pod -- bash -c "mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql"
```

### **Set a Secure Root Password**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your-secure-password';
```

---

## **Summary**

1. **MySQL container starts** (`Entrypoint script starts`).
2. **Database initialization** (`InnoDB starts`).
3. **Temporary MySQL server starts** (`Setting up system tables`).
4. **Warnings appear** (self-signed CA, missing time zone files).
5. **Temporary server shuts down** (`Initialization complete`).
6. **MySQL final startup** (`Running on port 3306`).

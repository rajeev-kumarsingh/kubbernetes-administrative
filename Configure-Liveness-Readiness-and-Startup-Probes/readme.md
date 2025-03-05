# Probes in Kubernetes

In Kubernetes, **probes** are used to determine the health and readiness of a container. These probes help Kubernetes decide whether to restart a container, remove it from service, or allow traffic to it.

## Types of Probes

There are three types of probes:

1. **Liveness Probe** – Checks if the container is still running.
2. **Readiness Probe** – Checks if the container is ready to accept traffic.
3. **Startup Probe** – Ensures that slow-starting applications are not killed prematurely.

---

## 1. Liveness Probe

- **Purpose**: Determines if the container is still alive. If the check fails, Kubernetes restarts the container.
- **Use Case**: If an application is stuck due to a deadlock but the process is still running, a liveness probe will detect this and restart the container.

### **Example: Liveness Probe in a Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: nginx
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 5
```

---

## 2. Readiness Probe

- **Purpose**: Determines if a container is ready to serve requests.
- **Use Case**: If an application requires some initialization before handling traffic (e.g., database connection setup), the readiness probe ensures that it does not receive traffic until it is fully ready.

### **Example: Readiness Probe in a Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: nginx
          readinessProbe:
            httpGet:
              path: /ready
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
```

---

## 3. Startup Probe

- **Purpose**: Ensures that slow-starting applications are given enough time to start.
- **Use Case**: If an application takes a long time to initialize (e.g., a Java application with a long startup time), Kubernetes might restart it before it has a chance to start properly. A startup probe prevents premature restarts.

### **Example: Startup Probe in a Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: startup-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: nginx
          startupProbe:
            httpGet:
              path: /startup
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 10
```

---

## 4. Real-World Application Example: Flask Web Application with Probes

Here is an example of a **Flask web application** running inside a Kubernetes deployment with all three types of probes.

### **Flask Application (app.py)**

```python
from flask import Flask
import time

app = Flask(__name__)
startup_time = time.time()

@app.route("/health")
def health():
    return "OK", 200

@app.route("/ready")
def ready():
    if time.time() - startup_time > 10:
        return "Ready", 200
    return "Not Ready", 503

@app.route("/startup")
def startup():
    if time.time() - startup_time > 15:
        return "Started", 200
    return "Starting", 503

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### **Kubernetes Deployment with Probes**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
        - name: flask-app
          image: myrepo/flask-app:latest
          ports:
            - containerPort: 5000
          startupProbe:
            httpGet:
              path: /startup
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 3
            failureThreshold: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 3
            periodSeconds: 5
```

### **Explanation:**

- The **Flask application** has three endpoints for health, readiness, and startup probes.
- The **startup probe** ensures the application starts completely before being considered healthy.
- The **readiness probe** waits 10 seconds before allowing traffic.
- The **liveness probe** runs every 5 seconds to check if the application is alive.

---

## Comparison Table

| Probe Type    | Purpose                                                  | When It Fails                                                             |
| ------------- | -------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Liveness**  | Checks if the container is running                       | The container is restarted                                                |
| **Readiness** | Checks if the container is ready for traffic             | The container is removed from service but not restarted                   |
| **Startup**   | Ensures slow-starting applications are given enough time | The container is restarted if it doesn’t start within the given threshold |

---

## Conclusion

Probes are essential for ensuring application stability in Kubernetes.

- Use **liveness probes** to restart unhealthy containers.
- Use **readiness probes** to prevent traffic from reaching an unready container.
- Use **startup probes** for slow-starting applications.

This example demonstrates a real-world application with Kubernetes probes.

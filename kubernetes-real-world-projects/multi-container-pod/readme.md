# Deploy a multi-container pod with:

- An nginx container serving static files (requests: 100m CPU, 128Mi memory, limits: 250m CPU, 256Mi memory).
- A sidecar container that logs HTTP requests to a file (busybox with infinite loop logging, requests: 50m CPU, 64Mi memory, limits: 100m CPU, 128Mi memory).

---

```sh
touch multi-container.sh
```

```py
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-pod
  labels:
    app: nginx-sidecar
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
    - name: log-sidecar
      image: busybox
      command: ["/bin/sh", "-c"]
      args:
        - while true; do
            cat /var/log/nginx/access.log;
            sleep 5;
          done;
      resources:
        requests:
          cpu: "50m"
          memory: "64Mi"
        limits:
          cpu: "100m"
          memory: "128Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
  volumes:
    - name: log-volume
      emptyDir: {}

```

OR

```py
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-pod
  labels:
    app: nginx-sidecar
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
    - name: log-sidecar
      image: busybox
      command: ["/bin/sh", "-c"]
      args:
        - while true; do
            tail -f /var/log/nginx/access.log;
          done;
      resources:
        requests:
          cpu: "50m"
          memory: "64Mi"
        limits:
          cpu: "100m"
          memory: "128Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: nginx-logs-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-sidecar
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```

`multicontainer-Pod-with-PVC`

```py
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-logs-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-pod
  labels:
    app: nginx-sidecar
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "250"
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx/nginx

  - name: log-sidecar
    image: busybox:latest
    command: [ "sh", "-c", "while true; do tail -f /var/log/nginx/access.log; done;" ]
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: nginx-logs-pvc
---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-sidecar
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: ClusterIP

```

OR
`Multi-container-with-pvc-and-probe`

```py
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-pod
  labels:
    app: nginx-sidecar
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
      livenessProbe:
        httpGet:
          path: /healthz
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 5
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30
        periodSeconds: 10

    - name: log-sidecar
      image: busybox
      command: ["/bin/sh", "-c"]
      args:
        - while true; do
            tail -f /var/log/nginx/access.log;
          done;
      resources:
        requests:
          cpu: "50m"
          memory: "64Mi"
        limits:
          cpu: "100m"
          memory: "128Mi"
      volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
      livenessProbe:
        exec:
          command: ["/bin/sh", "-c", "ps aux | grep tail | grep -v grep"]
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        exec:
          command: ["/bin/sh", "-c", "test -f /var/log/nginx/access.log"]
        initialDelaySeconds: 5
        periodSeconds: 10

  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: nginx-logs-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-sidecar
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```

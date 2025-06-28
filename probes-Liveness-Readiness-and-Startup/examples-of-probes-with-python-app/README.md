# Liveness Probe TCP Socket in Kubernetes

- Liveness Probe TCP Socket is a part of Kubernetes, thanks to which you can control the health of the pods. If it is possible to open in the container, the specified port of the container can be considered healthy, otherwise, the status failure will be returned.

## TCP Socket settings

- Liveness Probe restarts the container when the command returns a failure code. This feature also has some useful settings such as:

1. `initialDelaySeconds` – time after which Liveness Probe should start polling the endpoint
2. `periodSeconds` – the frequency of polling the endpoint
3. `timeoutSeconds` – time after which the timeout will expire.
4. `successThreshold` – the minimum number of successful attempts after which the probe will determine the correct operation of the container

5. `failureThreshold` – number of failed attempts after which the container will restart

## TCP Socket options:

1. `port` – port the liveness probe tries to open

- To show how it works, we will use two paths: positive and negative paths (returns failure).

### `Positive path:`

```yaml
# YAML example
# liveness-pod-example.yaml
#
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcpsocket
spec:
  containers:
    - name: liveness
      image: nginx
      ports:
        - containerPort: 80
      livenessProbe:
        tcpSocket:
          port: 80
        initialDelaySeconds: 2 #Default 0
        periodSeconds: 2 #Default 10
        timeoutSeconds: 1 #Default 1
        successThreshold: 1 #Default 1
        failureThreshold: 3 #Default 3
```

### Create pod:

```bash
kubectl apply -f iveness-pod-example.yaml
```

### Describe the pod:

```bash
kubectl describe pod liveness-tcpsocket

```

```bash
Restart Count:  0
.
.
.
Events:
  Type    Reason     Age        From                          Message
  ----    ------     ----       ----                          -------
  Normal  Scheduled  <unknown>  default-scheduler             Successfully assigned jenkins/liveness-tcpsocket to dcpoz-d-sou-k8swor2
  Normal  Pulling    3s         kubelet, dcpoz-d-sou-k8swor2  Pulling image "nginx"
  Normal  Pulled     1s         kubelet, dcpoz-d-sou-k8swor2  Successfully pulled image "nginx"
  Normal  Created    1s         kubelet, dcpoz-d-sou-k8swor2  Created container liveness
  Normal  Started    0s         kubelet, dcpoz-d-sou-k8swor2  Started container liveness
```

- Pod works great, no issue count because nginx exposes the port 80 by default (open the port).

### Delete the pod:

```bash
kubectl delete pod/liveness-tcpsocket

```

`or`

```bash
kubectl delete -f liveness-pod-example.yaml

```

## Negative path:

```yaml
# YAML example
# liveness-pod-example2.yaml
#
apiVersion: v1
kind: Pod metadata:
name: liveness-tcpsocket
spec: containers:
  - name: liveness
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      tcpSocket:
        port: 8888
    initialDelaySeconds: 2 #Default 0
    periodSeconds: 2 #Default 10
    timeoutSeconds: 1 #Default 1
    successThreshold: 1 #Default 1
    failureThreshold: 3 #Default 3

```

### Create pod:

```bash
kubectl create -f liveness-pod-example2.yaml

```

### Describe the pod:

```bash
kubectl describe pod liveness-tcpsocket

```

`O/P`

```bash
Restart Count: 1 # It will keep growing!
.
.
.
Events:
  Type     Reason     Age              From                          Message
  ----     ------     ----             ----                          -------
  Normal   Scheduled  <unknown>        default-scheduler             Successfully assigned jenkins/liveness-tcpsocket to dcpoz-d-sou-k8swor2
  Normal   Pulling    7s               kubelet, dcpoz-d-sou-k8swor2  Pulling image "nginx"
  Normal   Pulled     5s               kubelet, dcpoz-d-sou-k8swor2  Successfully pulled image "nginx"
  Normal   Created    4s               kubelet, dcpoz-d-sou-k8swor2  Created container liveness
  Normal   Started    4s               kubelet, dcpoz-d-sou-k8swor2  Started container liveness
  Warning  Unhealthy  0s (x2 over 2s)  kubelet, dcpoz-d-sou-k8swor2  Liveness probe failed: dial tcp 10.32.0.11:8888: connect: connection refused
```

- The pod is constantly restarting because nginx exposes the default port and the liveness probe is trying to monitor on port 8888 which is closed.

---

# Kubernetes liveness probe – Command Exec

- Liveness Probe Command Exec is an element in Kubernetes, thanks to which you can control the state of life of a counter in Pods using command inside containers. This option allows us to check, for example, the content of files, the existence of files and other options (available from the command level) that can give us information about the correct work of our container.

## Command Exec settings

- Liveness Probe restarts the container when the command returns a failure code. This feature also has some useful settings such as:

1. `initialDelaySeconds` – time after which Liveness Probe should start polling the endpoint
2. `periodSeconds` – the frequency of polling the endpoint
3. `timeoutSeconds` – time after which the timeout will expire.
4. `successThreshold` – the minimum number of successful attempts after which the probe will determine the correct operation of the container
5. `failureThreshold` – number of failed attempts after which the container will restart

### exec options:

1. command – list of commands (depend on image distribution) to do by liveness probe

```yaml
# YAML example
# liveness-pod-example.yaml
#
apiVersion: v1
kind: Pod
metadata:
  name: liveness-command-exec
spec:
  containers:
    - name: liveness
      image: nginx
      ports:
        - containerPort: 80
      livenessProbe:
        exec:
          command:
            - cat
            - /usr/share/nginx/html/index.html
        initialDelaySeconds: 2 #Default 0
        periodSeconds: 2 #Default 10
        timeoutSeconds: 1 #Default 1
        successThreshold: 1 #Default 1
        failureThreshold: 3 #Default 3
```

## Liveness Probe – example

### Create pod:

```bash
kubectl create -f liveness-pod-example.yaml

```

### Describe the pod:

```bash
kubectl describe pod liveness-command-exec

```

```bash
Restart Count:  0
.
.
.
Events:
  Type     Reason     Age              From                          Message
  ----     ------     ----             ----                          -------
  Normal   Scheduled  <unknown>        default-scheduler             Successfully assigned jenkins/liveness-command-exec to dcpoz-d-sou-k8swor2
  Normal   Pulling    100s             kubelet, dcpoz-d-sou-k8swor2  Pulling image "nginx"
  Normal   Pulled     97s              kubelet, dcpoz-d-sou-k8swor2  Successfully pulled image "nginx"
  Normal   Created    97s              kubelet, dcpoz-d-sou-k8swor2  Created container liveness
  Normal   Started    97s              kubelet, dcpoz-d-sou-k8swor2  Started container liveness
  Warning  Unhealthy  2s (x2 over 4s)  kubelet, dcpoz-d-sou-k8swor2  Liveness probe failed: cat: /usr/share/nginx/html/index.html: No such file or directory
```

- Pod works great, no issue count.

### To check if the probe works, remove index.html file from “liveness-command-exec” pod.

```bash
kubectl exec -it liveness-command-exec -- rm /usr/share/nginx/html/index.html

```

- Index.html file was removed, and the liveness probe tried to check (by cat) this file. File does not exist; hence, the container will restart. You can see the result below.

```bash
kubectl describe pod liveness-command-exec

```

`O/P`

```bash
Restart Count: 1
.
.
.
Events:
  Type     Reason     Age                From                          Message
  ----     ------     ----               ----                          -------
  Normal   Scheduled  <unknown>          default-scheduler             Successfully assigned jenkins/liveness-command-exec to dcpoz-d-sou-k8swor2
  Warning  Unhealthy  5s (x3 over 9s)    kubelet, dcpoz-d-sou-k8swor2  Liveness probe failed: cat: /usr/share/nginx/html/index.html: No such file or directory
  Normal   Killing    5s                 kubelet, dcpoz-d-sou-k8swor2  Container liveness failed liveness probe, will be restarted
  Normal   Pulling    4s (x2 over 105s)  kubelet, dcpoz-d-sou-k8swor2  Pulling image "nginx"
  Normal   Pulled     2s (x2 over 102s)  kubelet, dcpoz-d-sou-k8swor2  Successfully pulled image "nginx"
  Normal   Created    2s (x2 over 102s)  kubelet, dcpoz-d-sou-k8swor2  Created container liveness
  Normal   Started    2s (x2 over 102s)  kubelet, dcpoz-d-sou-k8swor2  Started container liveness

```

---

# Kubernetes Liveness Probe – HTTPRequest

- Liveness Probe HTTPRequest is an element in Kubernetes, thanks to which you can control the state of life of a counter in Pods using the HTTP protocol. The component allows you to send queries to a given endpoint in the container and infer whether the application is working correctly.

## Liveness Probe settings

- Liveness Probe restarts the container when the given endpoint returns a status higher than `399`. This feature also has some useful settings such as:

1. `initialDelaySeconds` – time after which Liveness Probe should start polling the endpoint
2. `periodSeconds` – the frequency of polling the endpoint
3. `timeoutSeconds` – time after which the timeout will expire.
4. `successThreshold` – the minimum number of successful attempts after which the probe will determine the correct operation of the container
5. `failureThreshold` – number of failed attempts after which the container will be restarted

## httpGet options:

1. `host` – Host name to connect to
2. `scheme` – Protocol type HTTP or HTTPS
3. `path` – Path to access on the HTTP/HTTPS server
4. `httpHeaders–` Custom headers to set in the request
5. `port` – Port number to access on the container

```yaml
# YAML example
# liveness-pod-example.yaml
#
apiVersion: v1
kind: Pod
metadata:
  name: liveness-request
spec:
  containers:
    - name: liveness
      image: nginx
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 2 #Default 0
        periodSeconds: 2 #Default 10
        timeoutSeconds: 1 #Default 1
        successThreshold: 1 #Default 1
        failureThreshold: 3 #Default 3
```

## Liveness Probe – example

### Create pod:

```bash
kubectl create -f liveness-pod-example.yaml

```

## Describe the pod:

```bash
kubectl describe pod liveness-request

```

`O/P`

```bash
Restart Count:  0
.
.
.
Events:
Type Reason Age From Message
---- ------ ---- ---- -------
Normal Scheduled <unknown> default-scheduler Successfully assigned example-dc/liveness-request to dcpoz-d-sou-k8swor3
Normal Pulling 4m45s kubelet, dcpoz-d-sou-k8swor3 Pulling image "nginx"
Normal Pulled 4m42s kubelet, dcpoz-d-sou-k8swor3 Successfully pulled image "nginx"
Normal Created 4m42s kubelet, dcpoz-d-sou-k8swor3 Created container liveness
Normal Started 4m42s kubelet, dcpoz-d-sou-k8swor3 Started container liveness
```

- Pod works great, no issue count.

## To check if liveness probe works, remove index.html file from “liveness-request” pod.

```bash
kubectl exec -it liveness-request -- rm /usr/share/nginx/html/index.html

```

- Index.html file was removed and the liveness probe tried to send a request to port 80, because the response status was not in the range from 200 to 399. Hence, the container will be restarted. You can see the result below.

```bash
kubectl describe pod liveness-request

```

```bash
Restart Count: 1
.
.
.
Events:
Type Reason Age From Message
---- ------ ---- ---- -------
Normal Scheduled <unknown> default-scheduler Successfully assigned example-dc/liveness-request to dcpoz-d-sou-k8swor3
Normal Pulling 9s (x2 over 11m) kubelet, dcpoz-d-sou-k8swor3 Pulling image "nginx"
Warning Unhealthy 9s (x3 over 13s) kubelet, dcpoz-d-sou-k8swor3 Liveness probe failed: HTTP probe failed with statuscode: 403
Normal Killing 9s kubelet, dcpoz-d-sou-k8swor3 Container liveness failed liveness probe, will be restarted
Normal Pulled 7s (x2 over 11m) kubelet, dcpoz-d-sou-k8swor3 Successfully pulled image "nginx"
Normal Created 7s (x2 over 11m) kubelet, dcpoz-d-sou-k8swor3 Created container liveness
Normal Started 7s (x2 over 11m) kubelet, dcpoz-d-sou-k8swor3 Started container liveness
```

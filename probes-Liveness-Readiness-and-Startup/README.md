# Configure Liveness, Readiness and Startup Probes

## Kubernetes has various types of probes

- Liveness probe
- Readiness probe
- Startup probe

---

## Liveness probe

- Liveness probe determine when to restart a container. For example `liveness probes` could catch a deadlock when an application is running but unable to make progress.
- If a containers fails it's liveness probe repeatedly, the kubelet restart the container.
- Liveness probe do not wait for readiness probes to succed. If you want to wait before executing a liveness probe, you can either define `initialDelaySeconds` or use a `startup probe`.
- The `kubelet` uses the liveness probe to know when to restart a container. For example, liveness probe could catch a deadlock, where an application is running but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.
- A common pattern for liveness probe is use the same low-cost HTTP endpoint as for readiness probes but with a higher failureThresold. This ensures that the pod is observed as not ready for some period of time before it is hard killed.

## Caution

Liveness probes can be a powerful way to recover from application failures, but they should be used with caution. Liveness probes must be configured carefully to ensure that they truly indicate unrecoverable application failure, for example a deadlock.

## Note

Incorrect implementation of liveness probes can lead to cascading failures. This results in restarting of container under high load; failed client requests as your application became less scalable; and increased workload on remaining pods due to some failed pods. Understand the difference between readiness and liveness probes and when to apply them for your app.

- Many applications running for long period of time
  eventually transition to brocken states, and cannot recover except by being restarted. Kubernetes provides to detect and remedy such situations.

# Exercise

- In this exercise you create a Pod that runs container based on `registry.k8s.io/busybox:1.27.2` image.

```py
apiVersion: v1
kind: Pod
metadata:
  name: liveness
  label:
    test: liveness
    name: liveness-exec
spec:
  containers:
    name: liveness-container
    image: registry.k8s.io/busybox:1.27.2
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5



```

- In the configuration file you can see the Pod has single `container`. The `periodSeconds` field specifies the kubelet should perform the liveness prob every 5 seconds. The `initialDelaySeconds` field tells the kubelet that it should wait 5 seconds before performing the first probe.
- To perform a probe, the kubelet executes the command `cat /tmp/healthy` in the target container. if the command succeds it returns 0, and the kubelet considered the container to be alive and healthy. if the command returns the non -zero values the kubelet kills the container and restarts it.
- When the container starts, it executes this command.

```sh
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600;
```

- For the first 30 seconds of container life, there is a /tmp/healthy file. So during first 30 seconds, the command `cat /tmp/healthy` returns success code. After 30 seconds `cat /tmp/healthy` returns a failure code.
- Create the Pod

```sh
kubectl apply -f liveness-check.yaml
```

within 30 seconds view the Pod events

```sh
kubectl describe pod/liveness-exec
```

Or

```sh
kubectl get events
```

The output indicates that no liveness probes have failed yet:
![alt text](image.png)
After 35 seconds, view the Pod events again:

```sh
kubectl get events
```

At the bottom of the output, there are messages indicating that the liveness probes have failed, and the failed containers have been killed and recreated.
![alt text](image-1.png)

Wait another 30 seconds, and verify that the container has been restarted:

```sh
kubectl get pod liveness-exec
```

The output shows that RESTARTS has been incremented. Note that the RESTARTS counter increments as soon as a failed container comes back to the running state:
![alt text](image-2.png)

---

## Define a liveness HTTP request

- Another kind of liveness probe uses an `HTTP GET` request. Here is the configuration file for Pod that runs a container based on `registry.k8s.io/e2e-test-images/agnhost` image

```py
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
  labels:
    test: liveness
spec:
  containers:
  - name: liveness-http-container
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custome-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

```

- In the configuration file, you can see that the Pod has single container.
- The `periodSeconds` field specifies that the kubelet should perform prob every 3 seconds.
- The `initialDelaySeconds` field tells the kubelet that it should wait 3 seconds before performing first probe.
- TO perform a probe, the kubelet sends the `HTTP GET` request to the server that is running in the container and listening on port 8080. If the handler for the server's `/healthz` path return a success code, the kubelet considers the container to be alive and healthy. If the handler returns a failure code, the kubelet kills the container and restart it.
- Any code greater than 200 and less than 400 indicates success. Any other code indicates failure.
- For the first 10 seconds the container is alive. The `/healthz` handler returns a status of 200. After that, the handler returns a status of 500.

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

var started = time.Now()

func main() {
    http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
        duration := time.Since(started) // More idiomatic than time.Now().Sub(started)
        if duration.Seconds() > 10 {
            w.WriteHeader(http.StatusInternalServerError)
            w.Write([]byte(fmt.Sprintf("error: %.2f seconds since start", duration.Seconds())))
        } else {
            w.WriteHeader(http.StatusOK)
            w.Write([]byte("ok"))
        }
    })

    fmt.Println("Server starting on :8080...")
    http.ListenAndServe(":8080", nil)
}

```

- The kubelet starts performing health checks 3 seconds after the container starts. So the first couple of health check will succed. But after 10 seconds, the health check will fail, and the kubelet will kill and restart the container.
-
- ***

## Readiness probe

- Readiness probe determine when a container is ready to start accepting traffic. This is useful when waiting for an application time consuming initial tasks, such as establishing network connections, loading files and warming catches.
- If a readiness probe returns a failed failed state, kubernetes remove the probes from all matching `service endpoints`.
- Readiness probes runs on a container during it's whole lifecycle.
- The kubelet uses readiness probes to know when a container is ready to start accepting traffic. One use of this signal is to control which Pods are used as backend for Services.
- A Pod is considered ready when it's `ready condition` is true.
- When a Pod is not `ready`, it is removed from service loadbalancer.
- A Pod's `Ready` condition is false when it's `Node's ready condition is not true,` when one of the Pod's `readinessGates` is false, or when atleast one of it's container is not ready.

---

## Startup probe

- A startup probe verifies whether the application within the container is started. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the `kubelet` before they are up and running.
- If such a probe is configured, it disable `liveness` and `readiness` checks until it succeds.
- This type of probe only executes at startup, unlike liveness and readiness probes, which are run periodically.

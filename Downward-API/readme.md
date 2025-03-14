# Downward API

- There are two ways to expose Pod and Container fields to a running container:
  1. Environment Variables
  2. As files that are populated by special volume type.
- Together, these two ways of exposing Pod and container fields are called the downward API.
- It is useful for a container to have information about itself, without being overly to Kubernetes.
- The `downward API` allows container to consume information about themselves or the cluster without using the Kubernetes client or API server.
- Expose Pod information to Containers through `environment variables`
  - Here we will see how a Pod can use use `environment variable ` to expose information about itself to containers running in the Pod, using the `downward api`
  - You can use environment variables to expose Pod fields, container fields or both.
  - In Kubernetes there are two ways to expose pod and container fields to a running container:
    1. Environment Variables
    2. Volume files
  - Together, these two ways of exposing Pod and container fields are called the `downward API`
  - As service is the primary mode of communication between containerized application managed by Kubernetes, it is helpful to be able to discover them at runtime.

# Use Pod fields as value for environment variables.

In this example, we will create a Pod that has one container, and you project Pod-level fields into the running container as environment variables.

```sh
touch dapi-envars-pod.yaml
```

```py
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-fieldref
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox:1.27.2
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never


```

> > Note: The fields in this example are Pod fields. They are not fields of the container in the Pod.

- Create the Pod:

```sh
kubectl apply -f dap-envars-pod.yaml
```

- Verify that the container in the Pod is running:

```sh
# If the new Pod isn't yet healthy, rerun this command a few times.
kubectl get pods

```

- View the container's logs:

```sh
kubectl logs pod/dapi-envars-fieldref

```

`Output:`

```sh
kubectl logs pod/dapi-envars-fieldref

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4

multi-node-cluster2-worker2
dapi-envars-fieldref
default
10.244.1.4
```

To see why these values are in the log, look at the command and args fields in the configuration file. When the container starts, it writes the values of five environment variables to stdout. It repeats this every ten seconds.

- Next, get a shell into the container that is running in your Pod:

```sh
kubectl exec -it pods/dapi-envars-fieldref -c test-container -- sh
```

`Output:`

```sh
kubectl exec -it pods/dapi-envars-fieldref -c test-container -- sh
/ # printenv
CLUSTERIP_SERVICE_SERVICE_HOST=10.96.2.243
BACKEND_SERVICE_PORT_5000_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
NODEPORT_SERVICE_PORT_80_TCP=tcp://10.96.20.30:80
HOSTNAME=dapi-envars-fieldref
SHLVL=1
HOME=/root
CLUSTERIP_SERVICE_SERVICE_PORT=80
BACKEND_SERVICE_SERVICE_HOST=10.96.33.57
CLUSTERIP_SERVICE_PORT=tcp://10.96.2.243:80
BACKEND_SERVICE_PORT_5000_TCP=tcp://10.96.33.57:5000
MY_POD_NAMESPACE=default
CLUSTERIP_SERVICE_PORT_80_TCP_ADDR=10.96.2.243
BACKEND_SERVICE_PORT=tcp://10.96.33.57:5000
BACKEND_SERVICE_SERVICE_PORT=5000
CLUSTERIP_SERVICE_PORT_80_TCP_PORT=80
CLUSTERIP_SERVICE_PORT_80_TCP_PROTO=tcp
TERM=xterm
NODEPORT_SERVICE_SERVICE_HOST=10.96.20.30
MY_POD_IP=10.244.1.4
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
MY_POD_SERVIVE_ACCOUNT=default
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NODEPORT_SERVICE_SERVICE_PORT=80
CLUSTERIP_SERVICE_PORT_80_TCP=tcp://10.96.2.243:80
NODEPORT_SERVICE_PORT=tcp://10.96.20.30:80
MY_NODE_NAME=multi-node-cluster2-worker2
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
NODEPORT_SERVICE_PORT_80_TCP_ADDR=10.96.20.30
PWD=/
KUBERNETES_SERVICE_HOST=10.96.0.1
NODEPORT_SERVICE_PORT_80_TCP_PORT=80
BACKEND_SERVICE_PORT_5000_TCP_ADDR=10.96.33.57
NODEPORT_SERVICE_PORT_80_TCP_PROTO=tcp
MY_POD_NAME=dapi-envars-fieldref
BACKEND_SERVICE_PORT_5000_TCP_PORT=5000
/ #
```

```sh
kubectl exec -it Pod/dapi-envars-fieldref -- sh

```

`Output:`

```sh
kubectl exec -it Pod/dapi-envars-fieldref -- sh
/ # printenv
CLUSTERIP_SERVICE_SERVICE_HOST=10.96.2.243
BACKEND_SERVICE_PORT_5000_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
NODEPORT_SERVICE_PORT_80_TCP=tcp://10.96.20.30:80
HOSTNAME=dapi-envars-fieldref
SHLVL=1
HOME=/root
CLUSTERIP_SERVICE_SERVICE_PORT=80
BACKEND_SERVICE_SERVICE_HOST=10.96.33.57
CLUSTERIP_SERVICE_PORT=tcp://10.96.2.243:80
BACKEND_SERVICE_PORT_5000_TCP=tcp://10.96.33.57:5000
MY_POD_NAMESPACE=default
CLUSTERIP_SERVICE_PORT_80_TCP_ADDR=10.96.2.243
BACKEND_SERVICE_PORT=tcp://10.96.33.57:5000
BACKEND_SERVICE_SERVICE_PORT=5000
CLUSTERIP_SERVICE_PORT_80_TCP_PORT=80
CLUSTERIP_SERVICE_PORT_80_TCP_PROTO=tcp
TERM=xterm
NODEPORT_SERVICE_SERVICE_HOST=10.96.20.30
MY_POD_IP=10.244.1.4
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
MY_POD_SERVIVE_ACCOUNT=default
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NODEPORT_SERVICE_SERVICE_PORT=80
CLUSTERIP_SERVICE_PORT_80_TCP=tcp://10.96.2.243:80
NODEPORT_SERVICE_PORT=tcp://10.96.20.30:80
MY_NODE_NAME=multi-node-cluster2-worker2
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
NODEPORT_SERVICE_PORT_80_TCP_ADDR=10.96.20.30
PWD=/
KUBERNETES_SERVICE_HOST=10.96.0.1
NODEPORT_SERVICE_PORT_80_TCP_PORT=80
BACKEND_SERVICE_PORT_5000_TCP_ADDR=10.96.33.57
NODEPORT_SERVICE_PORT_80_TCP_PROTO=tcp
MY_POD_NAME=dapi-envars-fieldref
BACKEND_SERVICE_PORT_5000_TCP_PORT=5000
/ #
```

---

# Use container fields as values for environment variables

## In previous example, we used information from Pod-level field as the value for environment vairiables. In the next exercise, we are going to pass fields that are part of the Pod definition, but taken from specific `container` rather than the Pod overall.

```py
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-resourcefieldref
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox:1.27.2
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_CPU_REQUEST MY_CPU_LIMIT;
          printenv MY_MEM_REQUEST MY_MEM_LIMIT;
          sleep 10;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
  restartPolicy: Never

```

- In this manifest, You can see 4 environment variables. The `env` field is an array of environment variable definitions. The first element in array specifies that `MY_CPU_REQUEST` environment variable, get its value from the `requests.cpu` of a container named `test-container`. Similarly, the other environment variables gets their values from the fields that are specific to this container
- Create the Pod

```sh
kubectl apply -f dapi-envars-container.yaml

```

`Output:`

```sh
kubectl apply -f dapi-envars-container.yaml
pod/dapi-envars-resourcefieldref created
```

- Verify that the container in the Pod is running:

```sh
kubectl get pods
```

`Output:`

```sh
kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
dapi-envars-fieldref           1/1     Running   0          23h
dapi-envars-resourcefieldref   0/1     Error     0          83s
```

`Let's troubleshoot the error`

```sh
kubectl describe pod/dapi-envars-resourcefieldref
```

`Output:`

```sh
kubectl describe pod/dapi-envars-resourcefieldref
Name:             dapi-envars-resourcefieldref
Namespace:        default
Priority:         0
Service Account:  default
Node:             multi-node-cluster2-worker2/172.19.0.4
Start Time:       Thu, 13 Mar 2025 04:48:28 +0530
Labels:           app=downward-api
Annotations:      <none>
Status:           Failed
IP:               10.244.1.5
IPs:
  IP:  10.244.1.5
Containers:
  test-container:
    Container ID:  containerd://105570abfbd5b5d7793c367a5a70722ead931e84ef2bd3f771ac3fc9ecb6207b
    Image:         registry.k8s.io/busybox:1.27.2
    Image ID:      registry.k8s.io/busybox@sha256:545e6a6310a27636260920bc07b994a299b6708a1b26910cfefd335fdfb60d2b
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
    Args:
      while true; do echo -en '\n'; printenv MY_CPU_REQUEST MY_CPU_LIMIT; printenv MY_MEM_REQUEST MY_MEM_LIMIT;
    State:          Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Thu, 13 Mar 2025 04:48:28 +0530
      Finished:     Thu, 13 Mar 2025 04:48:28 +0530
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     250m
      memory:  64Mi
    Requests:
      cpu:     125m
      memory:  32Mi
    Environment:
      MY_CPU_REQUEST:  1 (requests.cpu)
      MY_CPU_LIMIT:    1 (limits.cpu)
      MY_MEM_REQUEST:  33554432 (requests.memory)
      MY_MEM_LIMIT:    67108864 (limits.memory)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tg4w8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   False
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-tg4w8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m46s  default-scheduler  Successfully assigned default/dapi-envars-resourcefieldref to multi-node-cluster2-worker2
  Normal  Pulled     3m46s  kubelet            Container image "registry.k8s.io/busybox:1.27.2" already present on machine
  Normal  Created    3m46s  kubelet            Created container: test-container
  Normal  Started    3m46s  kubelet            Started container test-container
```

## Pod is failing with Exit Code: 2, meaning there is likely a syntax error in the command or an issue with environment variable expansion.

> > Issue:
> > Key Fixes:
> > ✔ Added done at the end of the while loop to prevent syntax errors.
> > ✔ Ensured printenv prints all variables in a single call for better readability.

## ✔ Added sleep 10 to prevent excessive logging and immediate termination.

- View the container's logs:

```sh
kubectl logs
```

`Output:`

```sh
kubectl logs pod/dapi-envars-resourcefieldref

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864
```

---

# Define Environment Variables for a Container

## When you create a Pod, You can set environment variables for the containers that runs in the Pod. To set environment variables, include the `env` or `envFrom` fields have different effects.

- `env`: allows you to set environment variable for a container, specifying a value directly for each variable that you name
- `envFrom`: allows you to set environment variables for a container by referencing either a `configmap or Secret` are set as environment variables for the containers. You can also specify a common prefix string.
- Environment variable names consist of letters, numbers, underscores, dots, or hyphens, but the first character cannot be a digit. If the RelaxedEnvironmentVariableValidation feature gate is enabled, all printable ASCII characters except "=" may be used for environment variable names.

---

# Expose Pod information to containers through files

- Here we will see, How a Pod can use `downwardapi volume` to expose information about itself

- In Kubernetes, there are two ways to expose Pod and container fields to a running container.

1. Environment Variables
2. Volume files

---

# Store Pod Fields

## In this part of exercise, we will create a Pod that has one container, we will project Pod-level fields into the running container as file.

```py
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: registry.k8s.io/busybox:1.27.2
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations


```

- In the manifest, you can see that the pod has a `downwardAPI` volume, and the container mount the volume at `etc/podinfo`
- Look at the `items` array under `downwardAPI` volume. The first element specifies that the value of the Pod's `metadata.labels` field should be stored in a file named `labels`. The second element specifies that the values of the Pod's `annotations` field should be stored in a file named `annotations`

> > Note:
> > The fields in this example are Pod fields. They are not fields of the container in the Pod.

- Create the Pod:

```sh
kubectl apply -f dapi-volume.yaml
```

```sh
kubectl logs pod/kubernetes-downwardapi-volume-example
```

`Output:`

```sh
kubectl logs pod/kubernetes-downwardapi-volume-example


cluster="multi-node-cluster2"
sh: missing ]]
zone="us-east-1"

cluster="multi-node-cluster2"
sh: missing ]]
zone="us-east-1"

cluster="multi-node-cluster2"
sh: missing ]]

```

---

# Store container fields

## In the preceding excercise, you made Pod-level fields accessible using the downward API. In the next exercise, we are going to pass field that are part of Pod definition, but taken from the specific container rather than the Pod overall.

```sh
touch dapi-volume-resources.yaml
```

```py
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example-2
spec:
  containers:
    - name: client-container
      image: registry.k8s.io/busybox:1.27.2
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          if [[ -e /etc/podinfo/cpu_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_limit; fi;
          if [[ -e /etc/podinfo/cpu_request ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_request; fi;
          if [[ -e /etc/podinfo/mem_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_limit; fi;
          if [[ -e /etc/podinfo/mem_request ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_request; fi;
          sleep 5;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "cpu_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.cpu
              divisor: 1m
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
              divisor: 1Mi
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
              divisor: 1Mi


```

- Create the Pod

```sh
kubectl apply -f dapi-volume-resources.yaml
```

- Get a shell into your container that is running in your Pod

```sh
kubectl exec -it kubernetes-downwardapi-volume-example-2 -- sh
```

- In your shell, view the cpu_limit file:

### Run this in a shell inside the container

cat /etc/podinfo/cpu_limit

- You can use similar commands to view the cpu_request, mem_limit and mem_request files.

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

# Important Links

[Downward API](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)

[environment-variable](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

```

```

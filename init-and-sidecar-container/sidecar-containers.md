# Sidecar Containers

- Sidecar container are the secondary container that runs along with the main application containers within the same Pod. These containers are used to enhance and extend the functionality of the primary **_app container_** by providing additional services or functionality such as logging, monitoring, security, or data synchronization, without directly altering the primary application code.
- Typically, you only have one app container in a Pod. For example, if you have web application that requires a local web server, the local webserver is a `sidecar` and the web application itself is the app container.
- K8s implements sidecar containers as a special case of `init containers`; sidecar containers remain running afrer Pod startup. This document uses the term `regular init containers` to clearly refers to containers that only run during Pod startup.
- Provided that your container has the `SidecarContainers` `feature gate` enabled( The feature is active by default since k8s v1.29), you can specify a `restartPolicy` for container listed in Pod's `initContainers` field. These restartable **_sidecar_** containers are independent from other init containers and from the main application container(s) within the same Pod. These can be `started`, `stopped` or `restarted` without affecting the main application container and other initContainers.
- You can also run a Pod with multiple containers that are not marked as `initContainer` or `sidecar` containers. This is appropriate if the containers within the Pods are required for the Pod to work overall, but you don't need to control which container `start` or `stop` first.

### Example application

## Here is an example of Deployment with two containers, one of which is a sidecar:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
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
          image: alpine:latest
          command:
            [
              "sh",
              "-c",
              "while true; do echo 'logging' >> /opt/logs.txt; sleep 1; done",
            ]
          volumeMounts:
            - name: data
              mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always
          command: ["sh", "-c", "tail -F /opt/logs.txt"]
          volumeMounts:
            - name: data
              mountPath: /opt
      volumes:
        - name: data
          emptyDir: {}
```

### Create the deployment

```bash
kubectl apply -f deployment-sidecar.yaml
```

`O/P`

```bash
kubectl apply -f deployment-sidecar.yaml
deployment.apps/myapp created
```

### List and verify the Pods

```bash
kubectl get pods
```

`````bash

kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-5ffc9bd7f5-6qh4j   2/2     Running   0          3m41s
myapp-5ffc9bd7f5-rtbdj   2/2     Running   0          3m41s
```

Now dectrive the Pod for it'sevent`

````sh
kubectl describe pod/myapp
```

### Describe the Deployments

```bash
kubectl describe deploy/myapp
```

`O/P`
```bash
kubectl describe deploy/myapp
Name:                   myapp
Namespace:              default
CreationTimestamp:      Sun, 29 Jun 2025 13:48:06 +0530
Labels:                 app=myapp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=myapp
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myapp
  Init Containers:
   logshipper:
    Image:      alpine:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      tail -F /opt/logs.txt
    Environment:  <none>
    Mounts:
      /opt from data (rw)
  Containers:
   myapp:
    Image:      alpine:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      while true; do echo 'logging' >> /opt/logs.txt; sleep 1; done
    Environment:  <none>
    Mounts:
      /opt from data (rw)
  Volumes:
   data:
    Type:          EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:     <unset>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   myapp-5ffc9bd7f5 (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  7m42s  deployment-controller  Scaled up replica set myapp-5ffc9bd7f5 from 0 to 2
```
`````

## Sidecar containers and Pod lifecycle

- If an init container is created with its `restartPolicy` set to `Always`, it will start and remain running during the entire life of the Pod. This can be helpful for running supporting services separated from the main application containers.
- If a readinessProbe is specified for this init container, its result will be used to determine the `ready` state of the Pod.
- Since these containers are defined as init containers, they benefit from the same ordering and sequential guarantees as regular init containers, allowing you to mix sidecar containers with regular init containers for complex Pod initialization flows.
- Compared to regular init containers, sidecars defined within `initContainers` continue to run after they have started. This is important when there is more than one entry inside `.spec.initContainers` for a Pod. After a sidecar-style init container is running (the kubelet has set the `started` status for that init container to true), the kubelet then starts the next init container from the ordered `.spec.initContainers` list. That status either becomes true because there is a process running in the container and no startup probe defined, or as a result of its `startupProbe` succeeding.

---

## Jobs with sidecar containers

- If you define a job that uses sidecar using Kubernetes-style init containers, the sidecar container in each Pod does not prevent the job from completing after the main container has finished.

```yaml
apiVersion: v1
kind: Job
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myjob
          image: alpine:latest
      command: ["sh", "-c", 'echo "logging" > /opt/logs.txt']
      volumeMounts:
        - name: data
          mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always
          command: ["sh", "-c", "tail -F /opt/logs.txt"]
          volumeMounts:
            - name: data
              mountPath: /opt
      restartPolicy: Never
      volumes:
        - name: data
          emptyDir: {}
```

## Differences from application containers

- Sidecar containers run alongside **_app containers_** in the same pod. However, they do not execute the primary application logic; instead, they provide supporting functionality to the main application.

- Sidecar containers have their own independent lifecycles. They can be started, stopped, and restarted independently of app containers. This means you can update, scale, or maintain sidecar containers without affecting the primary application.
- Sidecar containers share the same network and storage namespaces with the primary container. This co-location allows them to interact closely and share resources.
- From a Kubernetes perspective, the sidecar container's graceful termination is less important. When other containers take all allotted graceful termination time, the sidecar containers will receive the SIGTERM signal, followed by the SIGKILL signal, before they have time to terminate gracefully. So exit codes different from 0 (0 indicates successful exit), for sidecar containers are normal on Pod termination and should be generally ignored by the external tooling.

## Resource sharing within containers

Given the order of execution for init, sidecar and app containers, the following rules for resource usage apply:

- The highest of any particular resource request or limit defined on all init containers is the effective init request/limit. If any resource has no resource limit specified this is considered as the highest limit.
- The Pod's effective request/limit for a resource is the sum of pod overhead and the higher of:
  - the sum of all non-init containers(app and sidecar containers) request/limit for a resource
  - the effective init request/limit for a resource
- Scheduling is done based on effective requests/limits, which means init containers can reserve resources for initialization that are not used during the life of the Pod.
- The QoS (quality of service) tier of the Pod's effective QoS tier is the QoS tier for all init, sidecar and app containers alike.

Quota and limits are applied based on the effective Pod request and limit.

-
-
-
-
-
-
-
-
-

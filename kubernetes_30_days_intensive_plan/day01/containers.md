# Containers

Technology for packaging an application along with its runtime dependencies.

- The word **_container_** is an overloaded term. Whenever you use the word, check whether your audience uses the same definition.
- Each container that you run is repeatable; the standardization from having dependencies included means that you get the same behavior wherever you run it.
- Containers decouple applications from the underlying host infrastructure. This makes deployment easier in different cloud or OS environments.
- Each node in a Kubernetes cluster runs the containers that form the Pods assigned to that node. Containers in a Pod are co-located and co-scheduled to run on the same node.

---

## Container images

A [`container image`](https://kubernetes.io/docs/concepts/containers/images/) is a ready-to-run software package containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings.

- Containers are intended to be stateless and [`immutable`](https://glossary.cncf.io/immutable-infrastructure/): you should not change the code of a container that is already running. If you have a containerized application and want to make changes, the correct process is to build a new image that includes the change, then recreate the container to start from the updated image.

---

## Container runtimes

- A fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.
- Kubernetes supports container runtimes such as `containerd`, `CRI-O`, and any other implementation of the [`Kubernetes CRI (Container Runtime Interface)`](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).
- Usually, you can allow your cluster to pick the default container runtime for a Pod. If you need to use more than one container runtime in your cluster, you can specify the `RuntimeClass` for a Pod to make sure that Kubernetes runs those containers using a particular container runtime.
- You can also use `RuntimeClass` to run different Pods with the same container runtime but with different settings.

---

## Container Environment

The Kubernetes Container environment provides several important resources to Containers:

- A filesystem, which is a combination of an [`image`](https://kubernetes.io/docs/concepts/containers/images/) and one or more [`volumes`](https://kubernetes.io/docs/concepts/storage/volumes/).
- Information about the Container itself.
- Information about other objects in the cluster.

### `Container information`

- The **_hostname_** of a Container is the name of the Pod in which the Container is running. It is available through the `hostname` command or the `gethostname` function call in libc.
- The Pod name and namespace are available as environment variables through the [`downward API`](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/).
- User defined environment variables from the Pod definition are also available to the Container, as are any environment variables specified statically in the container image.

---

### `Cluster information`

- A list of all services that were running when a Container was created is available to that Container as environment variables. This list is limited to services within the same namespace as the new Container's Pod and Kubernetes control plane services.

- For a service named foo that maps to a Container named bar, the following variables are defined:

```
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port the service is running on>
```

- Services have dedicated IP addresses and are available to the Container via DNS, if [`DNS addon`](https://releases.k8s.io/v1.33.0/cluster/addons/dns/) is enabled.

---

# Container Lifecycle Hooks

## `Overview`

Analogous to many programming language frameworks that have component lifecycle hooks, such as Angular, Kubernetes provides Containers with lifecycle hooks. The hooks enable Containers to be aware of events in their management lifecycle and run code implemented in a handler when the corresponding lifecycle hook is executed.

## `Container hooks`

## There are two hooks that are exposed to Containers:

`PostStart`

- This hook is executed immediately after a container is created. However, there is no guarantee that the hook will execute before the container ENTRYPOINT. No parameters are passed to the handler.

---

`PreStop`

- This hook is called immediately before a container is terminated due to an API request or management event such as a liveness/startup probe failure, preemption, resource contention and others. A call to the `PreStop` hook fails if the container is already in a terminated or completed state and the hook must complete before the `TERM` signal to stop the container can be sent. The Pod's `termination grace period` countdown begins before the `PreStop` hook is executed, so regardless of the outcome of the handler, the container will eventually terminate within the Pod's termination grace period. No parameters are passed to the handler.

---

`StopSignal`

- The StopSignal lifecycle can be used to define a stop signal which would be sent to the container when it is stopped. If you set this, it overrides any `STOPSIGNAL` instruction defined within the container image.
- A more detailed description of termination behaviour with custom stop signals can be found in [`Stop Signals`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-stop-signals).

---

## `Hook handler implementations`

- Containers can access a hook by implementing and registering a handler for that hook. There are three types of hook handlers that can be implemented for Containers:

  - `Exec` - Executes a specific command, such as pre-stop.sh, inside the cgroups and namespaces of the Container. Resources consumed by the command are counted against the Container.
  - `HTTP` - Executes an HTTP request against a specific endpoint on the Container.
  - `Sleep` - Pauses the container for a specified duration. This is a `beta-level feature` default enabled by the `PodLifecycleSleepAction` f[`eature gate`](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/).
    > `Note`:
    > The beta level `PodLifecycleSleepActionAllowZero` feature gate which is enabled by default from `v1.33`. It allows you to set a sleep duration of zero seconds (effectively a no-op) for your Sleep lifecycle hooks.

---

## Hook handler execution

- When a Container lifecycle management hook is called, the Kubernetes management system executes the handler according to the hook action, `httpGet`, `tcpSocket` (deprecated) and `sleep` are executed by the **kubelet process**, and `exec` is executed in the container.
- The `PostStart` hook handler call is initiated when a container is created, meaning the container **ENTRYPOINT** and the `PostStart` hook are triggered simultaneously. However, if the `PostStart` **hook takes too long to execute or if it hangs, it can prevent the container from transitioning to a running state**.
- `PreStop` hooks are not executed asynchronously from the signal to stop the Container; the hook must complete its execution before the TERM signal can be sent. If a `PreStop` hook hangs during execution, the Pod's phase will be `Terminating` and remain there until the Pod is killed after its `terminationGracePeriodSeconds` expires. This grace period applies to the total time it takes for both the PreStop hook to execute and for the Container to stop normally. If, for example, `terminationGracePeriodSeconds` is 60, and the hook takes 55 seconds to complete, and the Container takes 10 seconds to stop normally after receiving the signal, then the Container will be killed before it can stop normally, since `terminationGracePeriodSeconds` is less than the total time (55+10) it takes for these two things to happen.
- If either a `PostStart` or `PreStop` hook fails, it kills the Container.
- Users should make their hook handlers as lightweight as possible. There are cases, however, when long running commands make sense, such as when saving state prior to stopping a Container.

---

## `Hook delivery guarantees`

- Hook delivery is intended to be **_at least once_**, which means that a hook may be called multiple times for any given event, such as for `PostStart` or `PreStop`. It is up to the hook implementation to handle this correctly.
- Generally, only single deliveries are made. If, for example, an HTTP hook receiver is down and is unable to take traffic, there is no attempt to resend. In some rare cases, however, double delivery may occur. For instance, if a kubelet restarts in the middle of sending a hook, the hook might be resent after the kubelet comes back up.

---

## Debugging Hook handlers

- The logs for a Hook handler are not exposed in Pod events. If a handler fails for some reason, it broadcasts an event. For `PostStart`, this is the `FailedPostStartHook` event, and for `PreStop`, this is the `FailedPreStopHook` event. To generate a failed `FailedPostStartHook` event yourself, modify the [`lifecycle-events.yaml`](lifecycle-events.yaml) file to change the `postStart` command to "`badcommand`" and apply it. Here is some example output of the resulting events you see from running kubectl describe pod lifecycle-demo:

```bash
Events:
  Type     Reason               Age              From               Message
  ----     ------               ----             ----               -------
  Normal   Scheduled            7s               default-scheduler  Successfully assigned default/lifecycle-demo to ip-XXX-XXX-XX-XX.us-east-2...
  Normal   Pulled               6s               kubelet            Successfully pulled image "nginx" in 229.604315ms
  Normal   Pulling              4s (x2 over 6s)  kubelet            Pulling image "nginx"
  Normal   Created              4s (x2 over 5s)  kubelet            Created container lifecycle-demo-container
  Normal   Started              4s (x2 over 5s)  kubelet            Started container lifecycle-demo-container
  Warning  FailedPostStartHook  4s (x2 over 5s)  kubelet            Exec lifecycle hook ([badcommand]) for Container "lifecycle-demo-container" in Pod "lifecycle-demo_default(30229739-9651-4e5a-9a32-a8f1688862db)" failed - error: command 'badcommand' exited with 126: , message: "OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: \"badcommand\": executable file not found in $PATH: unknown\r\n"
  Normal   Killing              4s (x2 over 5s)  kubelet            FailedPostStartHook
  Normal   Pulled               4s               kubelet            Successfully pulled image "nginx" in 215.66395ms
  Warning  BackOff              2s (x2 over 3s)  kubelet
```

---

# Define `postStart` and `preStop` handler

In this exercise, you create a Pod that has one Container. The Container has handlers for the postStart and preStop events.

Here is the configuration file for the Pod:

```bash
vim lifecycle-events-with-handler.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo-with-handler
spec:
  containers:
    - name: lifecycle-with-handler
      image: nginx
      lifecycle:
        postStart:
          exec:
            command:
              - /bin/sh
              - -c
              - echo "Hello from postStart handler" > /usr/share/message
        preStop:
          exec:
            command:
              - /bin/sh
              - -c
              - |
                nginx -s quit
                while killall -0 nginx; do
                  sleep 1
                done
```

- In the configuration file, you can see that the `postStart` command writes a `message` file to the Container's `/usr/share` directory. The `preStop` command shuts down nginx gracefully. This is helpful if the Container is being terminated because of a failure.
  `Create the Pod:`

```bash
kubectl apply -f lifecycle-events-with-handler.yaml
```

`Output:`

```bash
kubectl apply -f lifecycle-events-with-handler.yaml
pod/lifecycle-demo-with-handler created

```

```bash
kubectl get pods -o wide -w
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE                          NOMINATED NODE   READINESS GATES
lifecycle-demo                1/1     Running   0          19m   10.244.1.6   multi-node-cluster2-worker2   <none>           <none>
lifecycle-demo-with-handler   1/1     Running   0          6s    10.244.1.7   multi-node-cluster2-worker2   <none>           <none>
^C%
```

> Get a shell into the Container running in your Pod:

```bash
kubectl exec -it pods/lifecycle-demo-with-handler -- sh
```

`Output`:

```bash
kubectl exec -it pods/lifecycle-demo-with-handler -- sh
# ls
bin   dev		   docker-entrypoint.sh  home  media  opt   product_uuid  run	srv  tmp  var
boot  docker-entrypoint.d  etc			 lib   mnt    proc  root	  sbin	sys  usr
# cd usr
# ls
bin  games  include  lib  libexec  local  sbin	share  src
# cd share
# ls
X11		 bug		  debianutils  dpkg	   gdb	     libc-bin	  man	      misc	   perl5     terminfo	 zsh
base-files	 ca-certificates  dict	       fontconfig  info      libgcrypt20  maven-repo  nginx	   pixmaps   util-linux
base-passwd	 common-licenses  doc	       fonts	   java      lintian	  menu	      pam	   polkit-1  xml
bash-completion  debconf	  doc-base     gcc	   keyrings  locale	  message     pam-configs  tabset    zoneinfo
# cat message
Hello from postStart handler
#

```

---

> `Delete Pods`

```bash
kubectl delete pod/lifecycle-demo
pod "lifecycle-demo" deleted

kubectl delete pod/lifecycle-demo-with-handler
pod "lifecycle-demo-with-handler" deleted
```

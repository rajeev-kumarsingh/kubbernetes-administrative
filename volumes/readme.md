# Volumes

## Kubernetes volumes provide a way for container in Pods to access and share data via filesystem. There are different kind of volumes that you can use for different purposes, such as:

- Populating a configuration file based on a `ConfigMap` or `Secret`
- Providing some temprory scratch space for pod.
- Sharing a filesystem between two different containers in the same pod.
- Sharing a filesystem between two different pods(even if those pods run on different nodes)
- durably storing data so that it says available even if the pod restarts or is replaced.
- passing configuration information to an app running in a container, based on detail of the Pod the container is in(for example: telling a `sidecar container` what namespace the pod is running in)
- providing read only access to data in a different container image.

# Why volumes are important

- `Data Persistance`: On-disk files in a container are ephemeral, which present some problems for non-trival applications when running in containers. One problem occurs when container crashes or is stopped, the container state is not saved so all of the files that were created or modified during the lifetime of the container are lost. After a crash, kubelet restart the container with a clean state.
- `Shared storage`: Another problem occurs when multiple container are running in a `Pod` and need to share files. It can be challenging to set up and access a shared file system across all of the container.
- The Kubernetes volume abstraction can help you to solve both of these problems.

---

# How volumes works

- Kubernetes supports many types of volumes. A pod can use any number of volume type sumultaneously. `Ephemeral Volume ` types have a lifetime linked to a specific Pod, but `persistent volumes` exits beyond the lifetime of any individual pod. When a pod ceases to exist, Kubernetes delete ephemeral volume; however, Kubernetes does not destroy persistent volumes. For any kind of volume in a given pod, data is preserved across container restarts.
- To use a volume, specify the volumes to provide for the Pod in `.spec.volumes` and declare where to mount those volumes into containers in `.spec.containers[*].volumeMounts`.
- Volumes cannot mount within other volumes(but see [using subpath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath) for a related mechanism). Also, a volume cannot contain a hard link to anything in different volume.

---

# Types of volumes

## Kubernetes supports several types of volumes.

# `awsElasticBlockStore(deprecated)`: In Kunernets 1.32, all operations for the in-tree `awsElasticBlockStore` type are redirected to the `ebs.csi.aws.com` CSI driver.

- The AWSElasticBlockStore in-tree storage driver was deprecated in the the Kubernetes v1.19 release and then removed entirely in the v1.27 release.
- The Kubernetes projects suggests that you use the [AWS EBS](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)

# `azureDisk (deprecated)`

- In Kubernetes 1.32, all operations for the in-tree `azuredisk` type are redirected to the `disk.csi.azure.com` CSI driver
- The Azuredisk in-tree storage driver was deprecates in the Kubernetes v1.19 release and then removed entirely in the v1.27 release.
- The Kubernetes projects suggest that you use the [Azure disk](https://github.com/kubernetes-sigs/azuredisk-csi-driver) third party storage instead.

# `azureFile(deprecated)`

- The `azurefile` volume type mounts a Microsoft Azure File Volume(SMB 2.1 and 3.0) into a pod

# azureFile CSI Migration

- The `CSIMigration` feature for `azurefile`, when enabled, redirect all plugin operations from the existing in-tree plugin to the `file.csi.azure.com` Container Storage Interface(CSI) Driver. In order to use the feature, the `Azure File CSI Driver` must be installed on the cluster and the `CSIMigrationAzureFile` feature gate must be enabled.
- Azure File CSI driver does not support using the same volume with different fsgroups. If `CSIMigrationFile` is enabled using same volume with different fsgroup won't be supported at all.

# azureFile CSI Migration Complete

- To disable `azureFile` storage plugin from being loaded by the controller manager and the kubelete, set the `InTreePluginAzureFileUnregister` flag to `true`.

# cephfs(removed)

- Kubernetes 1.32 does not support `cephfs` volume type.
- The `cephfs` in-tree storage driver deprecated in the Kubernetes v1.11 release and then removed entirely in the v1.26 release.
- The Kubernetes projects suggest that you use the [openstack cinder](https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/cinder-csi-plugin/using-cinder-csi-plugin.md) third party storage driver instead.

---

# configMap

- A [configMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) provide a way to inject configuration data into pods. The data stored in the ConfigMap can be referenced in a volume of type `configMap` and the consumed by containerized applications running in a pod.
- When referencing a configMap, you provide the name of the ConfigMap in the volume. you can customize the path to use for a specific entry in the ConfigMap. The following configuration file shows how to mount the `log-config` ConfigMap onto a Pod called `configmap-pod`

```py
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level.conf
```

- The log-config ConfigMap is mounted as a volume, and all contents stored in its log_level entry are mounted into the Pod at path /etc/config/log_level.conf. Note that this path is derived from the volume's mountPath and the path keyed with log_level.

> > Note:

- You must create a [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-a-configmap) before you can use it.

- A ConfigMap is always mounted as readOnly.

- A container using a ConfigMap as a subPath volume mount will not receive updates when the ConfigMap changes.

- Text data is exposed as files using the UTF-8 character encoding. For other character encodings, use binaryData.

---

# downwardAPI

- The downwardAPI volume makes downwardAPI data available to applications. within the volume, you can find the exposed data as read only files in plain text format.

> > Note:
> > A container using the downward API as a subPath volume mount does not receive updates when field values change.

- The Downward API is a feature in Kubernetes that allows containers to access metadata about the pod and its environment without having to query the Kubernetes API server directly. This is useful for applications that need to know their own metadata, such as resource limits, pod names, namespaces, or labels.

## A Downward API volume is a special type of volume that mounts this metadata as files inside the container. This allows applications to access the information by reading files instead of using environment variables.

## How Downward API Volume Works

- When using the Downward API as a volume:

1. The metadata is exposed as files inside the container.
2. The application reads the metadata from the files at runtime.
3. The metadata includes:
   - Pod name
   - Namespace
   - Labels
   - Annotations
   - Resource limits and requests (CPU, Memory)

- Example: Using Downward API as a Volume

1. Create a Pod with Downward API Volume
   The following Kubernetes YAML file defines a pod that:

   - Mounts metadata (Pod name and Namespace) as files inside the container.
   - Mounts CPU and Memory limits as files.

```py
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-demo
  labels:
    app: myapp
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["/bin/sh", "-c", "cat /etc/podinfo/*; sleep 3600"]
    volumeMounts:
    - name: podinfo
      mountPath: /etc/podinfo
    - name: resources
      mountPath: /etc/resources
  volumes:
  - name: podinfo
    downwardAPI:
      items:
      - path: "pod_name"
        fieldRef:
          fieldPath: metadata.name
      - path: "namespace"
        fieldRef:
          fieldPath: metadata.namespace
  - name: resources
    downwardAPI:
      items:
      - path: "cpu_limit"
        resourceFieldRef:
          containerName: my-container
          resource: limits.cpu
      - path: "memory_limit"
        resourceFieldRef:
          containerName: my-container
          resource: limits.memory


```

- How to Apply and Verify

1. Deploy the Pod

- Run the following command to create the pod

```sh
kubectl apply -f downwardapi-volume.yaml
```

2. Check the Pod Logs

- Find the pod name

```sh
kubectl get pods

```

- Check the logs to see the metadata

```sh
kubectl logs downwardapi-demo

```

3. Check Files Inside the Container

- Open a shell inside the container

```sh
kubectl exec -it pods/downwardapi-demo -c my-container -- sh
```

```sh
kubectl exec -it pods/downwardapi-demo -c my-container -- sh
/ # ls -la
total 52
drwxr-xr-x    1 root     root          4096 Mar 16 21:54 .
drwxr-xr-x    1 root     root          4096 Mar 16 21:54 ..
drwxr-xr-x    2 root     root         12288 Sep 26 21:31 bin
drwxr-xr-x    5 root     root           360 Mar 16 21:54 dev
drwxr-xr-x    1 root     root          4096 Mar 16 21:54 etc
drwxr-xr-x    2 nobody   nobody        4096 Sep 26 21:31 home
drwxr-xr-x    2 root     root          4096 Sep 26 21:31 lib
lrwxrwxrwx    1 root     root             3 Sep 26 21:31 lib64 -> lib
dr-xr-xr-x  276 root     root             0 Mar 16 21:54 proc
-rw-r--r--    1 root     root            37 Mar 16 21:54 product_uuid
drwx------    1 root     root          4096 Mar 16 21:54 root
dr-xr-xr-x   11 root     root             0 Mar 16 21:45 sys
drwxrwxrwt    2 root     root          4096 Sep 26 21:31 tmp
drwxr-xr-x    4 root     root          4096 Sep 26 21:31 usr
drwxr-xr-x    1 root     root          4096 Mar 16 21:54 var
/ # cd etc/podinfo/
/etc/podinfo # ls -la
total 4
drwxrwxrwt    3 root     root           120 Mar 16 21:45 .
drwxr-xr-x    1 root     root          4096 Mar 16 21:54 ..
drwxr-xr-x    2 root     root            80 Mar 16 21:45 ..2025_03_16_21_45_59.1873752844
lrwxrwxrwx    1 root     root            32 Mar 16 21:45 ..data -> ..2025_03_16_21_45_59.1873752844
lrwxrwxrwx    1 root     root            16 Mar 16 21:45 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            15 Mar 16 21:45 pod_name -> ..data/pod_name
/etc/podinfo # cd ..command terminated with exit code 137

```

---

# emptyDir

- For a pod that defines an `emptyDir` volume, the volume is created when the pod is assign to a node. As the same says `emptyDir` volume is initially empty. All containers in the pod can read and write the same files in the `emptyDir` volume, though that volume can be mounted at the same or different paths in each container. When a Pod is removed from a node for any reason, the data in `emptyDir` is deleted permanentely.

> > > Note:

## A container crashing does not remove a Pod from a node. The data in an emptyDir volume is safe across container crashes.

## Uses for an emptyDir are:

- scratch space, such as for a disk-based merge sort
- checkpointing a long computation for recovery from crashes
- holding files that a content-manager container fetches while a webserver container serves the data
- The `emptyDir.medium` field controls where `emptyDir` volumes are stored. By default emptyDir volumes are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment. If you set the `emptyDir.medium` field to `"Memory"`, Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that, unlike disks, files you write count against the memory limit of the container that wrote them.
- A size limit can be specified for the default medium, which limits the capacity of the `emptyDir` volume. The storage is allocated from [node ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage). If that is filled up from another source (for example, log files or image overlays), the `emptyDir` may run out of capacity before this limit. If no size is specified, memory-backed volumes are sized to node allocatable memory.
- emptyDir configuration example

```py
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

- emptyDir memory configuration example

```py
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
      medium: Memory
```

---

1. How emptyDir Works

   - It is created when a Pod is assigned to a node.
   - The volume remains as long as the Pod is running.
   - When the Pod is deleted or moves to another node, the data is lost.
   - It can be stored either in memory (medium: "Memory") or on disk.

2. Why Use emptyDir?
   emptyDir is useful when:

   - Multiple containers in a Pod need to share temporary data.
   - You need temporary storage that persists across container restarts (but not Pod restarts).
   - You want to use memory (medium: "Memory") for high-speed temporary storage.

3. Example Use Cases

## Use Case

Shared Storage Between Containers
Multiple containers in a Pod can access the same emptyDir volume.

```py
apiVersion: v1
kind: Pod
metadata:
  name: shared-storage-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: [ "sh", "-c", "echo 'Hello from App' > /data/message.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: /data
      name: shared-volume
  - name: log-reader
    image: busybox
    command: [ "sh", "-c", "cat /data/message.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: /data
      name: shared-volume
  volumes:
  - name: shared-volume
    emptyDir: {}

```

Explanation:

- app-container writes data to /data/message.txt.
- log-reader reads the file from /data/message.txt.
- Both containers share the emptyDir volume.

## Use Case

2. High-Speed Storage Using Memory

- For performance-critical applications, emptyDir can store data in memory instead of disk.

```py
apiVersion: v1
kind: Pod
metadata:
  name: in-memory-storage
spec:
  containers:
  - name: fast-container
    image: busybox
    command: [ "sh", "-c", "echo 'Fast data' > /data/fast.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: /data
      name: memory-storage
  volumes:
  - name: memory-storage
    emptyDir:
      medium: "Memory"

```

Explanation:

- medium: "Memory" makes the volume store data in RAM instead of disk.
- Data is much faster to read/write but gets erased when the Pod is stopped.

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

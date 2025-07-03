
# 🧱 Configuring and Managing Nodes in KIND

This guide explains how to configure and manage nodes in a KIND (Kubernetes IN Docker) cluster, including node roles, images, port mappings, volumes, labels, taints, and kubeadm patches.

---

## 📌 Node Roles

KIND supports two types of nodes:

- **control-plane**: Runs the Kubernetes control plane components.
- **worker**: Joins as a worker node and runs application workloads.

---

## 🧾 Basic Cluster with One Control Plane and One Worker

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
```

---

## 🖼️ Specify Node Image

You can specify the Kubernetes version by selecting the KIND node image:

```yaml
nodes:
  - role: control-plane
    image: kindest/node:v1.29.0
  - role: worker
    image: kindest/node:v1.29.0
```

Find compatible images at: https://github.com/kubernetes-sigs/kind/releases

---

## 🌐 Extra Port Mappings (Host ↔️ Container)

Expose ports on the host to your cluster nodes:

```yaml
extraPortMappings:
  - containerPort: 30000
    hostPort: 8080
    protocol: TCP
```

Use case: Access NodePort services from your local machine.

---

## 📂 Extra Mounts (Volumes)

Mount host directories into nodes:

```yaml
extraMounts:
  - hostPath: /Users/rajeev/kind-vol
    containerPath: /mnt/kind-vol
```

Useful for local persistent volumes or sharing configs/scripts with nodes.

---

## 🏷️ Node Labels

Add Kubernetes labels to nodes at startup:

```yaml
labels:
  topology.kubernetes.io/zone: us-central1-a
  app: frontend
```

Use these for nodeSelectors, affinity rules, or topology awareness.

---

## ⛔ Node Taints

Prevent pods from scheduling on specific nodes:

```yaml
taints:
  - key: "dedicated"
    value: "control-plane"
    effect: "NoSchedule"
```

Common for isolating workloads or applying tolerations.

---

## ⚙️ kubeadmConfigPatches (Per Node)

Customize `kubeadm` init/join settings per node:

```yaml
kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
        authorization-mode: AlwaysAllow
```

This is powerful for advanced node-level configurations.

---

## 🧪 Full Example: Multi-node Cluster

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    image: kindest/node:v1.29.0
    extraPortMappings:
      - containerPort: 30000
        hostPort: 8080
    extraMounts:
      - hostPath: /tmp/data
        containerPath: /mnt/data
    labels:
      app: control
    taints:
      - key: "dedicated"
        value: "control"
        effect: "NoSchedule"
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
            authorization-mode: AlwaysAllow

  - role: worker
    image: kindest/node:v1.29.0
    labels:
      app: backend
```

---

## 🧹 Managing Nodes After Cluster Creation

Once the KIND cluster is running, use `kubectl`:

- List nodes:
  ```bash
  kubectl get nodes
  ```

- Describe a node:
  ```bash
  kubectl describe node <node-name>
  ```

- Cordon/drain a node:
  ```bash
  kubectl cordon <node-name>
  kubectl drain <node-name> --ignore-daemonsets
  ```

- Add taints/labels manually:
  ```bash
  kubectl label node <node-name> zone=us-central1-a
  kubectl taint nodes <node-name> dedicated=control:NoSchedule
  ```

---

## 📚 References

- [KIND Docs](https://kind.sigs.k8s.io/docs/)
- [kubeadm Configuration](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)

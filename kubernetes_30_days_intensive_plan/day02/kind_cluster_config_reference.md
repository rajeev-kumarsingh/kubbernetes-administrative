
# 🐳 KIND Cluster Configuration - Full Reference

This document lists all available fields in a KIND cluster configuration file (`kind.x-k8s.io/v1alpha4`), with descriptions and examples.

---

## 📄 Example Config

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

networking:
  disableDefaultCNI: false
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
  kubeProxyMode: "iptables"
  apiServerAddress: "127.0.0.1"
  apiServerPort: 6443

nodes:
  - role: control-plane
    image: kindest/node:v1.29.0
    extraPortMappings:
      - containerPort: 30000
        hostPort: 8080
        protocol: TCP
    extraMounts:
      - hostPath: /host/data
        containerPath: /container/data
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    labels:
      topology.kubernetes.io/zone: us-central1-a
    taints:
      - key: "dedicated"
        value: "control-plane"
        effect: "NoSchedule"

  - role: worker
    image: kindest/node:v1.29.0

featureGates:
  IPv6DualStack: true

runtimeConfig:
  "api/alpha": "true"

kubeadmConfigPatches:
  - |
    kind: ClusterConfiguration
    metadata:
      name: config
    apiServer:
      extraArgs:
        "authorization-mode": "Node,RBAC"
```

---

## ✅ Top-Level Fields

| Field                | Type     | Description                                 |
|---------------------|----------|---------------------------------------------|
| `kind`              | string   | Must be `"Cluster"`                         |
| `apiVersion`        | string   | Use `kind.x-k8s.io/v1alpha4`                |
| `nodes`             | list     | List of nodes and their roles/config        |
| `networking`        | object   | Networking options                          |
| `featureGates`      | object   | Enable Kubernetes feature gates             |
| `runtimeConfig`     | object   | API runtime settings                        |
| `kubeadmConfigPatches` | list | Cluster-wide kubeadm patches (YAML)         |

---

## 🔧 `networking` Fields

| Field               | Type     | Description                                  |
|--------------------|----------|----------------------------------------------|
| `disableDefaultCNI`| bool     | Disable KIND’s default CNI                   |
| `podSubnet`        | string   | CIDR for pod network                         |
| `serviceSubnet`    | string   | CIDR for services                            |
| `apiServerAddress` | string   | Bind API server to a host IP                 |
| `apiServerPort`    | int      | API server port                              |
| `kubeProxyMode`    | string   | `iptables` or `ipvs`                         |

---

## 🧱 `nodes` Fields

| Field                  | Type     | Description                                |
|-----------------------|----------|--------------------------------------------|
| `role`                | string   | `control-plane` or `worker`                |
| `image`               | string   | Node image (e.g. kindest/node:v1.29.0)     |
| `extraPortMappings`   | list     | List of port maps (host ⇄ container)       |
| `extraMounts`         | list     | Mount host paths inside the node container |
| `kubeadmConfigPatches`| list     | Inline kubeadm patches for this node       |
| `labels`              | object   | Custom Kubernetes node labels              |
| `taints`              | list     | Node taints (key, value, effect)           |

---

## ⚙️ Other Fields

### `featureGates`

Enable experimental Kubernetes features:

```yaml
featureGates:
  IPv6DualStack: true
```

### `runtimeConfig`

Enable or disable Kubernetes APIs:

```yaml
runtimeConfig:
  "api/alpha": "true"
```

---

## 📌 Notes

- Always use the correct `kindest/node` image version for your Kubernetes version.
- Use `kubeadmConfigPatches` carefully; incorrect patches may break cluster startup.
- KIND is best for **local development**, **CI**, or **testing**.

---

## 🔗 Useful Resources

- [KIND Docs](https://kind.sigs.k8s.io/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/)

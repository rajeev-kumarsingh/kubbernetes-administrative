# Day 24/40 - Kubernetes RBAC Continued - Clusterrole and Clusterrole Binding

![alt text](./images/image.png)

### Notes and key takeaways from the video

**Roles & Role Bindings: Granting Permissions**

Think of roles as job descriptions defining user or group actions. Role bindings link these roles to specific users or groups, granting them those permissions.

- **Roles:** Define permissions within a namespace.
  - Example: A "developer" role might allow getting, viewing, and deleting pods, as well as creating and viewing secrets.
- **Role Bindings:** Connect roles to users or groups.

**Cluster Roles: Expanding Permissions Beyond Namespaces**

Cluster roles are like superpowered roles that can access resources across all namespaces. This includes powerful stuff like managing nodes, persistent volumes (PVs), certificate signing requests (CSRs), and even namespaces.

- **Check Available Resources:** Use

```
kubectl api-resources namespaced=false
```

to see cluster-scoped resources. Replace `false` with `true` to check the namespace-scoped resources.

**Cluster Role Bindings: Linking Users to Cluster Roles**

Like role bindings, cluster role bindings connect users or groups to cluster roles, granting cluster-wide permissions.

### Commands to create cluster role and cluster role binding

https://kubernetes.io/docs/reference/access-authn-authz/rbac/

```
kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes
```

![alt text](./images/image-1.png)

#

```
kubectl create clusterrolebinding node-reader-binding --clusterrole=node-reader --user=rajeev
```

![alt text](./images/image-2.png)

#

```
kubectl get clusterrolebinding | grep node
```

![alt text](./images/image-3.png)

#

```
kubectl describe clusterrole/node-reader
```

![alt text](./images/image-4.png)

#

```
kubectl describe clusterrolebinding/node-reader-binding
```

![alt text](./images/image-5.png)

#

```
kubectl auth can-i get pod --as rajeev
```

![alt text](./images/image-6.png)

#

```
kubectl config get-contexts
kubectl config use-context rajeev
kubectl config get-contexts
```

![alt text](./images/image-7.png)

#

```
kubectl get pods
kubectl get no -o wide
kubectl auth whoami
```

![alt text](./images/image-8.png)

#

```
kubectl delete node/multinode-worker2
```

![alt text](./images/image-9.png)

#

#

**Key Points:**

- Roles are namespace-scoped, while cluster roles are cluster-wide.
- Use role bindings and cluster role bindings to assign permissions to users or groups.
- Carefully manage permissions to ensure security and prevent unauthorized access.

#

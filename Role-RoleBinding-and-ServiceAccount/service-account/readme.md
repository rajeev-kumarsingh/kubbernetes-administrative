# Day 25/40 - Kubernetes Service Account - RBAC Continued

### Service account in Kubernetes

![alt text](./images/image.png)

#### What is a service account in Kubernetes

There are multiple types of accounts in Kubernetes that interact with the cluster. These could be user accounts used by Kubernetes Admins, developers, operators, etc., and service accounts primarily used by other applications/bots or Kubernetes components to interact with other services.

- To list all service account

```
kubectl get sa
kubectl get sa -A # list service account with all the namespaces
kubectl get sa -A | grep default # List all the service account with default namespace
```

![alt text](./images/image-1.png)
![alt text](./images/image-2.png)
![alt text](./images/image-3.png)

#

- To describe service account with specific namespace

```
kubectl describe sa/default
```

![alt text](./images/image-4.png)

#

- To see the default service account in yaml format

```
kubectl get sa/default -o yaml
```

![alt text](./images/image-5.png)

#

- To redirect the default service account in yaml format

```
kubectl get sa/default -o yaml >default-sa.yaml
```

#

#

- To create a service account

```
kubectl create sa name
kubectl get sa
```

```
kubectl create sa build-sa
kubectl get sa
```

- To get build-sa service account in yaml and redirect it to a build-sa.yaml file

```
kubectl create sa build-sa --dry-run=client -o yaml >build-sa.yaml
```

![alt text](./images/image-7.png)
![alt text](./images/image-6.png)

#

# Manually create a long-lived API token for a ServiceAccount

```
vim secret.yaml
```

```
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-sa
type: kubernetes.io/service-account-token
```

## Apply secret.yaml

```
kubectl apply -f secret.yaml
```

![alt text](./images/image-8.png)

#

```
kubectl get secret
```

![alt text](./images/image-9.png)

#

```
kubectl describe secret/build-robot-secret
```

![alt text](./images/image-10.png)

#

```
kubectl get pods --as build-sa
```

![alt text](./images/image-11.png)

#

```
kubectl auth can-i get pod --as build-sa
```

![alt text](./images/image-12.png)

#

#

- Then you can add role and role binding to grant access

## Create a role

```
kubectl create role build-role \
--verb=list,get,watch \
--resource=pod,node
```

![alt text](./images/image-13.png)

#

```
kubectl get role
```

![alt text](./images/image-14.png)

#

## Now create a rolebinding

```
kubectl create rolebinding build-role-binding \
--role=build-role \
--user=build-sa
```

![alt text](./images/image-15.png)

#

```
kubectl get rolebinding
```

![alt text](./images/image-16.png)

#

```
kubectl auth can-i get pod --as build-sa
```

![alt text](./images/image-17.png)

#

```
kubectl get pods --as build-sa
```

![alt text](./images/image-18.png)

#

> Note: Kubernetes also create 1 default service account in each of the default namespace such as kube-sytem, kube-node-lease and so on

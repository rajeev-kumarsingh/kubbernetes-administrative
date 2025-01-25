# Secrets

- A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
- Such information might otherwise be put in a Pod specification or in a container image.
- Using a Secret means that you don't need to include confidential data in your application code.
- Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods.
- Kubernetes, and applications that run in your cluster, can also take additional precautions with Secrets, such as avoiding writing sensitive data to nonvolatile storage.
- Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.

#

# Caution:

Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment.

## In order to safely use Secrets, take at least the following steps:

1. [Enable Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets.
2. [Enable or configure RBAC rules](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) with least-privilege access to Secrets.
3. Restrict Secret access to specific containers.
4. [Consider using external Secret store providers](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#provider-for-the-secrets-store-csi-driver).

#

# Uses for Secrets

You can use Secrets for purposes such as the following:

- [Set environment variables for a container.](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
- [Provide credentials such as SSH keys or passwords to Pods](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#provide-prod-test-creds)
- [Allow the kubelet to pull container images from private registries](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- The Kubernetes control plane also uses Secrets; for example, [bootstrap token Secrets](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) are a mechanism to help automate node registration.

#

# Before you begin

- You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)
- To do this exercise, you need the docker command line tool, and a [Docker ID](https://docs.docker.com/accounts/create-account/) for which you know the password.

#

- Secrets are namespaced object, that is exist in the context of the namespace.
- You can access them via a Volume or an environment variable from a container running in a Pod.
- The Secrets data on nodes is stored in **tmpfs volume** (tmpfs is a file system which keeps all files in virtual memory, Everything in the tmpfs is temporary in the sense that no file will be created on your hard drive)
- A per-secret size of 1Mib exist.
- The **API-SERVER** store **secrets** as plain text in **etcd**.
- Secrets can be created
  1. from a text file
  2. from a yaml file

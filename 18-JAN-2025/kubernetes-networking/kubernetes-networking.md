# Kubernetes Networking, services, NodePort and Volumes

Kubenetes Networking addresses four concerns

1. Container within Pod use networking to communicate via loopback.
2. Cluster networking provides communication between different Pods.
3. The service resources lets you expose an application runnint on a Pod to be reachable from outside your cluster.
4. You can also use services to publish services only for consumption inside your cluster container to container communication on same pod happen through localhost within the container.

# Service Type: `ClusteIP`

- Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an [`Ingress`](https://kubernetes.io/docs/concepts/services-networking/ingress/) or a [`Gateway`](https://gateway-api.sigs.k8s.io/).
- This default Service type assigns an IP address from a pool of IP addresses that your cluster has reserved for that purpose.
- If you define a Service that has the `.spec.clusterIP` set to "None" then Kubernetes does not assign an IP address. See [`headless Services`](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) for more information.

## Choosing your own IP address

- You can specify your own cluster IP address as part of a Service creation request. To do this, set the `.spec.clusterIP` field. For example, if you already have an existing DNS entry that you wish to reuse, or legacy systems that are configured for a specific IP address and difficult to re-configure.
- The IP address that you choose must be a valid `IPv4` or `IPv6` address from within the `service-cluster-ip-range` `CIDR` range that is configured for the API server. If you try to create a Service with an invalid `clusterIP` address value, the API server will return a `422` HTTP status code to indicate that there's a problem.
- Read [`avoiding collisions`](https://kubernetes.io/docs/reference/networking/virtual-ips/#avoiding-collisions) to learn how Kubernetes helps reduce the risk and impact of two different Services both trying to use the same IP address.

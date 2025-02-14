KUBERNETES SERVICES
Service is a method for exposing Pods in your cluster.
Each Pod gets its own IP address But we need to access from IP of the Node..
If you want to access pod from inside we use Cluster-IP.
If the service is of type NodePort or LoadBalancer, it can also be accessed.
from outside the cluster.
It enables the pods to be decoupled from the network topology, which makes
it easier to manage and scale applications

TYPES
CLUSTER-IP
NODE PORT
LOAD BALANCER

TYPES OF SERVICES
ClusterIP: A ClusterIP service provides a stable IP address and DNS name for
pods within a cluster. This type of service is only accessible within the cluster
and is not exposed externally.
NodePort: A NodePort service provides a way to expose a service on a static
port on each node in the cluster. This type of service is accessible both within
the cluster and externally, using the node's IP address and the NodePort.
LoadBalancer: A LoadBalancer service provides a way to expose a service
externally, using a cloud provider's load balancer. This type of service is
typically used when an application needs to handle high traffic loads and
requires automatic scaling and load balancing capabilities.
ExternalName: This is a similar object service to ClusterIP but it does have
DNS CName instead of Selectors and labels. In other words, services will be
mapped to a DNS name. You can view the Service YML file and see how to use
this service.

COMPONENTS OF SERVICES
A service is defined using a Kubernetes manifest file that describes its properties
and specifications. Some of the key properties of a service include:
Selector: A label selector that defines the set of pods that the service will
route traffic to.
Port: The port number on which the service will listen for incoming traffic.
TargetPort: The port number on which the pods are listening for traffic.
Type: The type of the service, such as ClusterIP, NodePort, LoadBalancer, or
ExternalName.

Labels, Selectors, and Node Selectors:
Labels:
Labels are used to organize Kubernetes Objects such as Pods, nodes, etc.
You can add multiple labels over the Kubernetes Objects.
Labels are defined in key-value pairs.
Labels are similar to tags in AWS or Azure where you give a name to filter
the resources quickly.
You can add labels like environment, department or anything else
according to you.

Label-Selectors:
Once the labels are attached to the Kubernetes objects those objects will
be filtered out with the help of labels-Selectors known as Selectors.
The API currently supports two types of label-selectors equality-based
and set-based. Label selectors can be made of multiple selectors that are
comma-separated.

Node Selector:
Node selector means selecting the nodes. Node selector is used to choose the
node to apply a particular command.
This is done by Labels where in the manifest file, we mentioned the node label
name. While running the manifest file, master nodes find the node that has
the same label and create the pod on that container.
Make sure that the node must have the label. If the node doesn’t have any
label then, the manifest file will jump to the next node.

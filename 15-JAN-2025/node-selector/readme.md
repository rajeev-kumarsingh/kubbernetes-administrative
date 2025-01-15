# Node Selector
- One use case for selecting labels is to constrain the set of nodes onto which a pod can schedule. i.e you can tell a pod to only schedule/run on a specified/particular node.
- Generally, such constraints are unnecessary, as the scheduler will automatically do a reasonable placement, but on certain circumstances we might need it.
- we can use label to tag nodes.
- You the nodes are tagged, you can use the ___label-selector___ to specify the pods run only on specific nodes.
- First we give lable to the node.
- Then use node-selector to the configuration of the pod.
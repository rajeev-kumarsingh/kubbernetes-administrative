# Labels and Selectors 
- Labels are the mechanism you use to organise kubernetes objects.
- A label is a key value pair without any predefined meaning that can be attached to the objects.
- labels are similar to __tag__ in __AWS__ or __Git__ where you use a name to quick reference.
- You are free to choose __labels__ as you need it to refer an __environment__ which is used for __dev__ or __Testing__ or __Production__, refer a product group like DepartmentA, DepartmentB
- Multiple labels can be added to a single object.

E.g,
```
vim pod.yaml

```
```
apiVersion: v1
kind: Pod
metadata: 
  name: label-example
  labels:
    env: Production
    class: pods
spec:
  containers:
  - name: label-example
    image: nginx
    command: ["bin/bash", "-c", "while true; do echo Hello Rajeev; sleep 5; done"]
```
Apply the pod.yaml file
```
kubectl apply -f pod.yaml
```
Check labels of Pods 
```
kubectl get pods --show-labels
```

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
    command: ["bin/bash", "-c", "while true; do echo "Hello Rajeev"; sleep 5; done"]
```
Apply the pod.yaml file
```
kubectl apply -f pod.yaml
```
Output:
![alt text](image.png)
Check labels of Pods 
```
kubectl get pods --show-labels
```
Output :
![alt text](image-1.png)
#
 Add label to an existing pod through __imperative way__  
 ```
 kubectl label pod label-example env=test
 ```
 ![alt text](image-2.png)
 #
 Now List Pods matching a label
 ```
 kubectl get pods -l env=Production
 ```
 ![alt text](image-3.png)
 As we can see __key__, __pair__ of labels are case sensitive

#
Now list pods which lables are not Production
```
kubectl get pods -l env!=Production
```
```
kubectl get pods --show-labels
```
OutPut:
![alt text](image-4.png)

We can select multiple lables 
```
kubectl get pods -l env!=Production,env!=test
```
```
kubectl get pods --show-labels
```
OutPut:
![alt text](image-5.png)
#
 ___Delete Pod using labels___
 - check pods labels
 ```
 kubectl get pods --show-labels
 ```
 Output:
 ![alt list-pods-with-labels](image-6.png)
- Now let's delete pods with labels env=test and run=nginx
```
kubectl delete pods -l env=test
kubectl delete pods -l run=nginx
kubectl get pods --show-labels
```
Output:
![alt delete pod using label](image-7.png)

# Labels-Selectors
- Unlike ___name/UIDs___, labels do not provides uniqueness, as in general, we can expect many objects to carry the same label.
- Once ___label___ are attaches to an ___OBJECT___, we would need a filter to narrow down and these are called ___Selectors___.
- The ___api___ currently supports two types of ___Selectors___ 
>> 1. Equality based: = and !=
    - E.g., 
    name: rajeev  
    class: node
    class: pod
    env: dev
    env: test
    env: prod
>> 2. Set Based: in, notin and exists 
    - E.g., 
      env in (Production, dev)
      env notin (Production, dev)
      env exists (Production, dev)
- A label selector can be made of multiple requirements which are comma-separated
```
kubectl get <resource> --selector=<label-key>=<label-value>
```
1. Basic Example: Filter by a single label
```
kubectl get pods \
--selector=env=Production
```
OutPut:
![kubectl get pods --selector=env=Production (Filter by a single label)](image-8.png)
This lists all pods with the label **env=Production**

2. Multiple Labels: Logical AND To match resources with multiple labels, separate the labels with commas (__,__):
```
kubectl get pods --selector=env=production,app=nginx

```
OutPut:
![Logical AND To match resources with multiple labels (kubectl get pods --selector=env=production,app=nginx) ](image-9.png)
This lists pods with both _env=production_ and _app=nginx_.
All the given label should available on single object or resources or else it will give output as No resources found in default(current) namespace, We can mention namespace as well with -n <namespace-name> to find in specific namespace.

3. Filter by Key Existence To filter resources that have a specific label key, use:
```
kubectl get pods --selector='env'

```
OutPut:
![Filter by Key Existence(╰─ kubectl get pods --selector='env')](image-10.png)
This lists pods that have the env label (regardless of its value).

4. Negation: Exclude a Label Use ! to exclude resources with a specific label:
```
kubectl get pods --selector='!env'
```
OutPut:
![Negation: Exclude a Label Use ! to exclude resources with a specific label: kubectl get pods --selector='!env'](image-11.png)
This lists pods that do not have the env label.

5. Filter by Label Value Inequality To filter resources where a label value does not match, use !=
```
kubectl get pods --selector='env!=Production'
kubectl get pods --show-labels

```
OutPut:
![Filter by Label Value Inequality To filter resources where a label value does not match, use != (ubectl get pods --selector='env!=Production')](image-12.png)
This lists pods where the env label exists but its value is not production.

6. Filter All Resources by Label You can use the --all-namespaces flag to search across all namespaces:
```
kubectl get pods --selector=env=Production -all-namespaces
```
OutPut:
![Search across all namespaces](image-13.png)
List pods along-with labels across all namesapaces 
```
kubectl get pods --show-labels --all-namespaces
```
OutPut:
![List Pod with namespace across all namespaces](image-14.png)
List all namespaces
```
kubectl get ns
```
OutPut:
![List all namespaces](image-15.png)
# Advance Options
1. Output formats
- Wide Format:
```
kubectl get pods --selector=env=Production -o wide
kubectl get pods -o wide
kubectl get pods --show-labels -o wide
```
Output:
![wide format](image-16.png)
2. JSON or YAML:
```
kubectl get pods --selector=env=Production -o json
kubectl get pods --selector=env=Production -o yaml

```
This returns the yaml file of matching label pods 
We can save those yaml or json file 
```
kubectl get pods --selector=env=Production -o json > pod.json
kubectl get pods --selector=env=Production -o yaml > pod.yaml
ls -la
```
Output:
![alt text](image-17.png)
# Where to Use --selector
The --selector flag works with most kubectl commands, including:
- kubectl get
- kubectl delete
- kubectl describe
- kubectl logs

# Set Based Selectors
Set-based selectors in Kubernetes are used to filter resources based on whether labels meet certain set operations, such as inclusion (in), exclusion (notin), or existence (exists, !exists). These selectors provide more advanced filtering capabilities compared to equality-based selectors.
## Syntax
```
kubectl get <resource> --selector='<label-key> <operation> (<values>)'
```
### Supported Operations
1. ___in___: Matches if the label key exist and it's value is one of the specified values.

```
kubectl get pods --selector='env in (Production, test)'

kubectl get pods --selector='env in (Production, test)' -o wide

kubectl get pods --show-labels -o wide
```
Output:
![kubectl get pods --selector='env in (Production, test)'](image-18.png)
List pods where the env label is either Production or test

2. Filter by Exclusion (notin)
List pods where the env label is neither Production nor test:
```
kubectl get pods --show-labels -o wide 

kubectl get pods --selector='env notin (Production, test) -o wide'
```
Output:
![notin selector: kubectl get pods --selector='env notin (Production, test)'](image-19.png)

3.  Filter by Existence (exists)
>> List pods that have the env label, regardless of its value:

```
kubectl get pods --selector='env'
```
Or explicitly:
```
kubectl get pods --selector='env exists'
```
Output:
![kubectl get pods --selector='env'](image-20.png)

4. Filter by Non-Existence (!exists)
List pods that do not have the env label:
```
kubectl get pods --selector='!env'
```
Or explicitly:
```
kubectl get pods --selector='env !exists'
```
Output:
![!exists (kubectl get pods --selector='!env')](image-21.png)

5. Combine Set-Based Selectors
>>
You can combine multiple set-based selectors by separating them with commas. For example, list pods where:
- The env label is either production or test.
- The tier label exists.

### set the tier label on specified pod
check on which pod we have to set tier label
```
kubectl get pods --show-labels
```
Output
![list pods with labels](image-22.png)

let's set tier label on nginx pod
```
kubectl label pod nginx  tier=frontend
```
Output:
![label pod (kubectl label pod nginx tier=frontend)](image-23.png)

```
kubectl get pods --show-labels
```
Output:
![List pods with labels (--show-labels)](image-24.png)

Now let's use combine multiple set selector
```
kubectl get pods --selector='env in (Production, test),tier '
```
Output:
![combine multiple set based selector](image-25.png)


```
kubectl get pods --selector='env in (Production, test),!tier '
```
Output:
![combine multiple set based selector using comma(,)](image-26.png)

6. Set-Based Selectors Across Namespaces
> To filter resources across namespaces, add the --all-namespaces flag:

```
kubectl get pods --all-namespaces --selector='env in (Production, test)'
```
Output:
![set based seletor across all namespaces](image-27.png)

```
kubectl get pods --all-namespaces --selector='env notin (Production, test)'
```
Output:
![set based seletor used notin across all namespaces](image-28.png)

## Where to Use Set-Based Selectors
Set-based selectors can be used in any kubectl command that supports the --selector flag, such as:

- kubectl get
- kubectl delete
- kubectl describe

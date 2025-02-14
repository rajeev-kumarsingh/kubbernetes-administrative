To get all the pods: kubetctl get pods (or) kubectl get pod (or) kubectl get po
To delete a pod: kubectl delete pod pod_name
To get IP of a pod: kubectl get po pod_name -o wide
To get IP of all pods: kubectl get po -o wide
To get all details of a pod: kubectl describe pod podname
To get all details of all pods: kubectl describe po
To get the pod details in YAML format: kubectl get pod pod-1 -o yaml
To get the pod details in JSON format: kubectl get pod pod-1 -o json
To enter into a pod: kubectl exec -it pod_name -c cont_name bash
To get the logging info of our pod: kubectl logs pod_name
To get the logs of containers inside the pod: kubectl logs pod_name -c cont-name

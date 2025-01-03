Sample commands used

Create Horizontal Pod Autoscaling
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

To increase the load on the application
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

kubectl get hpa php-apache --watch

kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

Sample YAML used
1.Deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: php-apache
spec:
selector:
matchLabels:
run: php-apache
template:
metadata:
labels:
run: php-apache
spec:
containers: - name: php-apache
image: registry.k8s.io/hpa-example
ports: - containerPort: 80
resources:
limits:
cpu: 500m
requests:
cpu: 200m

---

apiVersion: v1
kind: Service
metadata:
name: php-apache
labels:
run: php-apache
spec:
ports:

- port: 80
  selector:
  run: php-apache

2. hpa.yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
   name: php-apache
   spec:
   scaleTargetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: php-apache
   minReplicas: 1
   maxReplicas: 10
   metrics:

- type: Resource
  resource:
  name: cpu
  target:
  type: Utilization
  averageUtilization: 50

Horizontal Pod Autoscaling(HPA):
Horizontal Pod Autoscaling (HPA) in Kubernetes dynamically adjusts the number of pods in a deployment, replica set, or stateful set based on observed CPU, memory, or custom metrics. This ensures that your application can scale to handle varying workloads efficiently while optimizing resource utilization.

How HPA Works
1.Metrics Collection:

The Kubernetes Metrics Server gathers resource usage data, such as CPU and memory.
Alternatively, custom metrics can be defined using the Custom Metrics API.

2.Scaling Decision:

HPA compares the current metrics to a target threshold you set.
If the usage exceeds the threshold, HPA increases the pod count. If it falls below, HPA decreases it.

3.Pod Adjustments:

HPA updates the replica count in the corresponding resource (e.g., Deployment) to scale up or down.

Setup Steps

1. Enable Metrics Server:
   i. Ensure the Metrics Server is installed in your cluster.
   ii. You can deploy it using the Metrics Server documentation.
2. Apply HPA to a Deployment: Create or update your deployment with resource requests and limits defined. For example:
   apiVersion: apps/v1
   kind: Deployment
   metadata:
   name: example-deployment
   spec:
   replicas: 2
   template:
   spec:
   containers: - name: example-container
   image: nginx
   resources:
   requests:
   cpu: 200m
   limits:
   cpu: 500m

3. Create an HPA Resource: Use the kubectl autoscale command or a YAML manifest:
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
   name: example-hpa
   spec:
   scaleTargetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: example-deployment
   minReplicas: 1
   maxReplicas: 10
   metrics:

- type: Resource
  resource:
  name: cpu
  target:
  type: Utilization
  averageUtilization: 50

4. Apply the HPA:
   kubectl apply -f hpa.yaml

Monitor and Verify HPA

1. Check HPA Status:
   kubectl get hpa
2. View Detailed Information:
   kubectl describe hpa example-hpa
3. Simulate Load: Use a load-testing tool (e.g., kubectl run, Apache Benchmark, or wrk) to simulate traffic and observe how the HPA scales pods.

Custom Metrics Example
To use custom metrics (e.g., request count):

1. Set up the Custom Metrics API with a compatible metrics provider (e.g., Prometheus Adapter).
2. Modify the HPA definition to include custom metrics:
   metrics:

- type: Pods
  pods:
  metric:
  name: http_requests_per_second
  target:
  type: AverageValue
  averageValue: 50

Vertical Pod AutoScaling:
Vertical Pod Autoscaling (VPA) in Kubernetes automatically adjusts the resource requests and limits (CPU and memory) for pods in a deployment or stateful set based on observed resource usage. This ensures that pods are appropriately sized for their workloads without manual intervention.

How Vertical Pod Autoscaling Works

1. Metrics Collection:
   The Kubernetes Metrics Server or external monitoring tools (like Prometheus) gather CPU and memory usage data.
2. Recommendations:
   i.VPA calculates resource requests and limits based on historical and current usage.
   ii.Pods are recreated with the recommended values during updates.
3. Modes: VPA can operate in three modes:
   i. Off: Only generates recommendations; no action is taken.
   ii.Auto: Automatically adjusts resources and restarts pods.
   iii.Initial: Adjusts resource requests only during pod creation.

Use Cases
Applications with fluctuating resource requirements.
Avoid over-provisioning or under-provisioning resources.
Applications where resizing is acceptable via pod restarts.

Setting Up VPA

1. Install VPA:
   Deploy the Vertical Pod Autoscaler components using the VPA official manifests.
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/vertical-pod-autoscaler/deploy/vpa-release.yaml
   2.Create a VPA Object: Define a VerticalPodAutoscaler resource for your deployment:
   apiVersion: autoscaling.k8s.io/v1
   kind: VerticalPodAutoscaler
   metadata:
   name: example-vpa
   spec:
   targetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: example-deployment
   updatePolicy:
   updateMode: "Auto" # Options: "Off", "Initial", or "Auto"
2. Apply the VPA:
   kubectl apply -f vpa.yaml
   4.Monitor Recommendations:
   kubectl get vpa
   kubectl describe vpa example-vpa

Modes Explained
Off: Only shows recommendations; no changes are made to pods.
updatePolicy:
updateMode: "Off"

Initial: Adjusts requests and limits only at pod creation time.
updatePolicy:
updateMode: "Initial"

Auto: Continuously adjusts resources and restarts pods if necessary.
updatePolicy:
updateMode: "Auto"

Benefits
Optimized resource utilization.
Reduced risk of OOM (Out of Memory) errors.
Automatic tuning of resource requests and limits for better performance.

Limitations
Restarts are required for resource updates (can disrupt stateless workloads; stateful workloads require more care).
Not suitable for scenarios where constant uptime is critical.
Might not handle rapid, short-term workload spikes well.

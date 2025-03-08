# Real-World Kubernetes Hands-on Project

## **Project: Deploying a Scalable E-Commerce Application on Kubernetes**

### **Overview**

This project simulates a real-world Kubernetes use case where we deploy a microservices-based e-commerce application using **Flask (Backend), React (Frontend), PostgreSQL (Database)**, and integrate it with AWS EKS and Application Load Balancer (ALB). We will also set up **CI/CD with CircleCI** and implement **monitoring and security best practices**.

---

## **1. Project Architecture**

### **Components:**

- **Frontend**: React application served using Nginx.
- **Backend**: Flask-based REST API.
- **Database**: PostgreSQL as a stateful component.
- **Kubernetes Cluster**: AWS EKS.
- **Load Balancer**: AWS ALB for routing traffic.
- **CI/CD**: CircleCI for automated deployments.
- **Monitoring**: Prometheus, Grafana, Loki, and Fluentd.
- **Security**: RBAC, Network Policies, Trivy for vulnerability scanning.

---

## **2. Setting Up Kubernetes Cluster**

### **Step 1: Create an AWS EKS Cluster**

Use Terraform to provision an EKS cluster.

```bash
terraform init
terraform apply -auto-approve
```

Once created, configure `kubectl` to interact with the cluster:

```bash
aws eks update-kubeconfig --region <region> --name <cluster-name>
```

### **Step 2: Deploy PostgreSQL (Database Layer)**

Create a StatefulSet for PostgreSQL.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          env:
            - name: POSTGRES_USER
              value: "admin"
            - name: POSTGRES_PASSWORD
              value: "password"
```

Apply the deployment:

```bash
kubectl apply -f postgres.yaml
```

### **Step 3: Deploy Flask Backend**

Create a Kubernetes deployment for the Flask API.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
        - name: flask
          image: your-dockerhub-user/flask-backend:latest
          ports:
            - containerPort: 5000
```

Apply the deployment:

```bash
kubectl apply -f flask-backend.yaml
```

### **Step 4: Deploy React Frontend**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:
      containers:
        - name: react
          image: your-dockerhub-user/react-frontend:latest
          ports:
            - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f react-frontend.yaml
```

### **Step 5: Configure AWS ALB for Traffic Routing**

Create an Ingress with AWS ALB annotations:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - host: my-ecommerce-app.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-frontend
                port:
                  number: 80
```

Apply the ingress:

```bash
kubectl apply -f ingress.yaml
```

---

## **3. Implementing CI/CD with CircleCI**

### **Step 1: Configure `.circleci/config.yml`**

```yaml
version: 2.1
jobs:
  build-and-deploy:
    docker:
      - image: circleci/python:3.8
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Build and Push Flask Image
          command: |
            docker build -t your-dockerhub-user/flask-backend .
            docker push your-dockerhub-user/flask-backend
      - run:
          name: Deploy to Kubernetes
          command: |
            kubectl set image deployment/flask-backend flask=your-dockerhub-user/flask-backend:latest
workflows:
  version: 2
  deploy:
    jobs:
      - build-and-deploy
```

### **Step 2: Commit and Push to GitHub**

```bash
git add .
git commit -m "Added CI/CD pipeline"
git push origin main
```

---

## **4. Monitoring with Prometheus & Grafana**

### **Step 1: Install Prometheus & Grafana**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

Access Grafana:

```bash
kubectl port-forward svc/grafana 3000:80
```

Log in to Grafana using `admin/prom-operator` and configure dashboards.

---

## **5. Implementing Security Best Practices**

### **Step 1: Restrict Pod Communication with NetworkPolicies**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector:
    matchLabels:
      app: flask
  policyTypes:
    - Ingress
    - Egress
```

```bash
kubectl apply -f network-policy.yaml
```

### **Step 2: Scan Images with Trivy**

```bash
trivy image your-dockerhub-user/flask-backend
```

---

## **6. Summary & Next Steps**

Congratulations! 🎉 You have:

- **Deployed a microservices app** on Kubernetes.
- **Implemented CI/CD** with CircleCI.
- **Set up monitoring** with Prometheus & Grafana.
- **Applied security best practices** with Network Policies & Trivy.

### **Next Steps:**

✅ Implement Service Mesh (Istio/Linkerd) for traffic management.
✅ Auto-scale using HPA & Cluster Autoscaler.
✅ Use Terraform for full infrastructure automation.

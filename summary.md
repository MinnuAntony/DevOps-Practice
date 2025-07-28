# Final Deployment: Complete Notes


## ✪ Goal:

To **deploy a containerized app** (e.g., Spring Boot) to a **Kubernetes cluster**, integrating everything you've learned:

* Docker (image build/push)
* Kubernetes (Pods, Deployment, Service, ConfigMaps, Secrets, HPA)
* GitHub Actions (CI/CD)

---

## 🔹 1. Docker Essentials

### ✅ Build Docker Image

```bash
docker build -t <dockerhub-username>/<app-name>:<tag> .
```

* Builds an image using the Dockerfile in the current directory.

### ✅ Push to Docker Hub

```bash
docker login
docker push <dockerhub-username>/<app-name>:<tag>
```

* Pushes the image to your Docker registry so Kubernetes can pull it.

---

## 🔹 2. GitHub Actions (CI/CD Pipeline)

### ✅ Sample GitHub Actions Workflow: `.github/workflows/docker-ci.yml`

```yaml
name: Docker CI

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker
      uses: docker/setup-buildx-action@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and Push Docker Image
      run: |
        docker build -t <your-dockerhub-username>/<app-name>:latest .
        docker push <your-dockerhub-username>/<app-name>:latest
```

Store DockerHub credentials in GitHub **Secrets**:

* `DOCKER_USERNAME`
* `DOCKER_PASSWORD`

---

## 🔹 3. Kubernetes Concepts & YAML

> Kubernetes abstracts infrastructure to manage containers at **scale**.

### 📆 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: <your-image>
```

* Smallest unit in K8s. Wraps around a container.
* Not scalable by itself.

### 📌 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: <your-image>
        ports:
        - containerPort: 8080
```

* Manages **replicas** of Pods and rolling updates.
* Ideal for production deployments.

### 🌐 Service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

* Exposes Pods to other apps or the outside world.
* Other types: `ClusterIP`, `LoadBalancer`

### ⚙️ ConfigMap

```bash
kubectl create configmap my-config --from-literal=ENV=prod
```

Use in Deployment:

```yaml
env:
- name: ENV
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: ENV
```

### 🔐 Secret

```bash
kubectl create secret generic my-secret --from-literal=DB_PASSWORD=secret123
```

Use in Deployment:

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: DB_PASSWORD
```

### 📈 Horizontal Pod Autoscaler (HPA)

```bash
kubectl autoscale deployment myapp-deployment --cpu-percent=50 --min=1 --max=5
```

* Automatically scales pods based on CPU usage.

---

## 🔹 4. Kubernetes Commands Cheatsheet

### 🟢 Apply YAML

```bash
kubectl apply -f deployment.yaml
```

### 🔍 View Resources

```bash
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl get configmaps
kubectl get secrets
```

### 🔧 Debugging

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### ❌ Delete Resources

```bash
kubectl delete -f deployment.yaml
```

---



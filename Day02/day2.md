# 📘 Kubernetes Day 2 —

## 🧠 Goal

Learn Kubernetes setup, components, and basic tools in a beginner-friendly way with real-life examples.

---

## ⚙️ Cluster Setup — Local vs Production

### 🧪 Local Setup (For Practice and Learning)

#### 1. Kind (Kubernetes IN Docker)

* Think of Kind as your personal Kubernetes lab.
* Runs Kubernetes inside Docker containers.
* Requires minimal system resources.
* Ideal for learning, testing, and CI/CD pipelines.
* ⚠️ Not suitable for production.

```bash
kind create cluster --name my-cluster
```

#### 2. Kubeadm

* Used to manually create Kubernetes clusters.
* Great for learning how clusters are built step by step.
* Offers full control but requires more setup time.

---

### ☁️ Cloud Production Setup

In production, use managed Kubernetes services from cloud providers:

| Cloud Provider | Kubernetes Service |
| -------------- | ------------------ |
| AWS            | EKS                |
| Google Cloud   | GKE                |
| Azure          | AKS                |

---

### 🏠 On-Premise Production Setup

* **OpenShift by RedHat**

  * Popular for on-premise clusters.
  * Includes UI, security features, and CI/CD support.

> 💡 **Tip:** Start with Kind or Kubeadm. Move to EKS, GKE, or AKS for production.

---

## 🧠 Kubernetes Core Concepts

### 🔧 Kubernetes Components (Behind the Scenes)

* `kube-apiserver`: Handles API requests (the receptionist).
* `etcd`: Stores cluster state (the database).
* `kube-scheduler`: Assigns pods to nodes.
* `kubelet`: Manages containers on each node.
* `kube-proxy`: Manages networking.

### 📦 Kubernetes Objects (You Create These)

* `Pod`: Runs your application.
* `Service`: Exposes your app.
* `ConfigMap`: Stores configuration.
* `Ingress`: Routes external traffic.

```bash
kubectl api-resources
```

### 🔁 Kubernetes Versioning

* Use fixed versions like `1.29.2`, avoid `latest`.
* Control plane must be the same or newer than nodes.

```bash
kubectl version --short
```

> ⚠️ Avoid using `latest` in production to prevent breaking changes.

---

## 🐳 Container & Image Basics

### 📦 What’s Inside a Container?

* App code (e.g., app.py)
* Dependencies (e.g., Flask)
* Minimal OS (if needed)
* Runtime (e.g., Python, Node.js)

### 🏷️ Use Tags for Versions

✅ `nginx:1.25.0`
❌ `nginx:latest`

### 🗂️ Where Images Are Stored

* Docker Hub
* GitHub Container Registry
* AWS ECR, Google Artifact Registry

### 📦 Base Images (Build Smarter)

| Base Image   | Use Case                       |
| ------------ | ------------------------------ |
| `scratch`    | Empty, smallest possible image |
| `alpine`     | Tiny Linux (\~5MB)             |
| `slim`       | Lightweight official image     |
| `distroless` | No shell, more secure          |

### 🛠️ Multi-Stage Builds

```dockerfile
# Build stage
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main

# Final stage
FROM gcr.io/distroless/static
COPY --from=builder /app/main /
ENTRYPOINT ["/main"]
```

✅ `distroless` is ideal for production.

---

## 🔁 Stateless vs Stateful Apps

### 🟢 Stateless Apps

* Don’t store data between restarts.
* Easy to scale.
* Examples: NGINX, Flask APIs.

### 🔵 Stateful Apps

* Store data across restarts.
* Examples: MySQL, MongoDB.
* Need Persistent Volumes and StatefulSets.

> 🛑 Best Practice: Use managed DBs like AWS RDS instead.

---

## 🧑‍💻 kubectl — Your Kubernetes Command Tool

Common Commands:

```bash
kubectl get nodes
kubectl get pods
kubectl get services
kubectl describe pod <name>
kubectl get events
```

> ⚠️ Avoid aliases like `k=kubectl` in team environments.

### 👀 Use k9s for a Visual Terminal UI

```bash
k9s
```

* Real-time pod status, logs, and debugging.
* 📌 [Get it here](https://github.com/derailed/k9s)

---

## 🧬 Kubernetes Architecture (Simple View)

```
Cluster → Node → Pod → Container
```

* **Cluster**: The entire Kubernetes system.
* **Node**: A physical or virtual machine.
* **Pod**: The smallest deployable unit.
* **Container**: Runs your actual app.

---

## 🧪 Sample Pod YAML (Beginner Example)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25.0
      ports:
        - containerPort: 80
```

### ✍️ YAML Explanation

| Field           | Description                 |
| --------------- | --------------------------- |
| `apiVersion`    | API version used (e.g., v1) |
| `kind`          | Object type (e.g., Pod)     |
| `metadata.name` | Unique name for the pod     |
| `spec`          | Defines the pod’s contents  |
| `containers`    | List of containers          |
| `image`         | Image to use                |
| `ports`         | Exposed ports               |

> 📌 Pods are **ephemeral**. Use **Deployments** to ensure high availability.

---

## 📋 Quick Recap

| Topic         | Best Practice                      |
| ------------- | ---------------------------------- |
| Local Cluster | Use Kind or Kubeadm for practice   |
| Prod Setup    | Use EKS, GKE, or AKS               |
| Image Tags    | Use fixed versions                 |
| Base Images   | Use alpine or distroless           |
| kubectl       | Use full commands, avoid aliases   |
| UI Tools      | Use k9s                            |
| Pod YAML      | Start with Pod, move to Deployment |

---


> 🚀 Stay consistent, practice daily, and move to hands-on real-world deployments!


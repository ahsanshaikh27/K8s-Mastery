🚀 Kubernetes Day 1: Mastering the Architecture

Welcome to your Day 1 journey into Kubernetes (K8s), the ultimate platform for orchestrating containerized applications! This README is your guide to understanding Kubernetes architecture, breaking down its core components in a beginner-friendly yet in-depth way. Whether you're just starting or refreshing your knowledge, let’s dive into the world of K8s with clarity and excitement! 🎉

🌟 What I Will Learn
By the end of this guide, I Will:

Grasp the Kubernetes architecture (control plane and worker nodes).
Understand key components like kube-apiserver, etcd, scheduler, and more.
See how Kubernetes ensures your apps are always running as desired.
Explore real-world examples to connect theory to practice.

Kubernetes is all about automation—think of it as a super-smart manager for your containerized apps!


What is Kubernetes?

Kubernetes (often called K8s) is an open-source platform that automates the deployment, scaling, and management of containerized applications. It’s like a conductor for your containers, ensuring they work together harmoniously across servers.
Why Kubernetes Rocks? 🎸

Portable: Runs anywhere—cloud, on-premises, or hybrid.

Self-Healing: Restarts failed containers or moves them to healthy nodes.

Scalable: Grows or shrinks your app based on demand.

Declarative: You define the desired state, and K8s makes it happen.


📌 Fun Fact: "Kubernetes" is Greek for "helmsman" or "pilot"—fitting for a system that steers your apps!


Kubernetes Architecture

Kubernetes is a distributed system split into two main parts:

Control Plane (Master Nodes): The brain that manages the cluster.

Worker Nodes: The muscle that runs your applications.

Here’s a quick visual:
📊 Kubernetes Cluster

├── 🧠 Control Plane
│   ├── kube-apiserver
│   ├── etcd
│   ├── kube-scheduler
│   └── kube-controller-manager
└── 💪 Worker Nodes
    ├── kubelet
    ├── kube-proxy
    └── container runtime

Control Plane

The control plane keeps your cluster in check, making high-level decisions.



Component

What It Does

kube-apiserver

The front door for all cluster interactions (e.g., via kubectl).
etcd

Stores the cluster’s state like a reliable database.

kube-scheduler

Places pods on the best nodes based on resources and policies.


kube-controller-manager

Runs controllers to keep the cluster’s desired state (e.g., ReplicaSet).


Worker Nodes

Worker nodes are where your apps live and run.



Component

What It Does

kubelet

Ensures containers in pods are healthy and running.


kube-proxy

Handles networking and load balancing for services.


Container Runtime

Runs containers (e.g., containerd, CRI-O).


Add-ons

Add-ons supercharge Kubernetes with extra features:

Networking (CNI): Plugins like Calico or Flannel for pod-to-pod communication.

Storage (CSI): Plugins like AWS EBS for persistent storage.

Monitoring: Tools like Prometheus for tracking cluster health.


Core Components Explained

Let’s zoom in on each component with real-world examples to make things crystal clear! 🔍

Kube-API Server

The kube-apiserver is the gateway to Kubernetes, handling all requests.
What It Does:

Authenticates users/services (via tokens, certificates, etc.).

Authorizes actions (e.g., can you delete a pod?).

Admission Control:
Mutating: Tweaks requests (e.g., sets default memory limits).
Validating: Blocks invalid requests (e.g., pods without labels).



Real-World Example:

run kubectl apply -f app.yaml to deploy a pod. The API server checks your credentials, ensures you’re allowed to create pods, and applies a 500Mi memory limit before saving the pod to etcd.


⚠️ Key Insight: The kube-apiserver is the only component that talks directly to etcd.

ETCD

etcd is the distributed key-value store that holds the cluster’s state.
Key Features:

Distributed: Runs across multiple nodes for reliability.
Raft Consensus: Ensures data consistency with leader election.
Write-Ahead Logging: Saves changes before applying them.

Real-World Example:

You deploy an app with 3 replicas. If a node crashes, etcd remembers the desired state, so Kubernetes can recreate the missing pods on another node.

Controllers

Controllers watch the cluster and ensure the actual state matches the desired state.
Popular Controllers:

ReplicaSet: Maintains the desired number of pod replicas.
Node Controller: Monitors node health.
CronJob: Schedules recurring tasks.

Real-World Example:

Your app needs 5 pod replicas, but a node failure leaves only 2 running. The ReplicaSet controller detects this and spins up 3 new pods.

Scheduler

The kube-scheduler decides which nodes run your pods.
How It Works:

Filtering: Finds nodes that meet pod requirements (e.g., CPU, GPU).

Scoring: Ranks nodes (e.g., least-loaded node wins).

Binding: Assigns the pod to the chosen node.

Real-World Example:

A pod needs 2 vCPUs and a GPU. The scheduler filters out non-GPU nodes, scores the remaining ones, and places the pod on the least-busy GPU node.


💡 Tip: Use taints, tolerations, or affinity rules to influence scheduling.

Kubelet

The kubelet is the agent on each node, ensuring pods and containers are running.
What It Does:

Talks to the kube-apiserver.

Manages container lifecycle via the container runtime.
Runs static pods from /etc/kubernetes/manifests.

Real-World Example:

A container crashes due to a memory leak. The kubelet restarts it based on the pod’s restartPolicy.

Kube-Proxy

kube-proxy handles networking magic, enabling service discovery and load balancing.
What It Does:

Maintains iptables or IPVS rules.
Routes traffic to the right pod endpoints.

Real-World Example:

Your frontend pod calls a backend service via its ClusterIP. Kube-proxy forwards the request to one of the backend pods, balancing the load.


Standard Interfaces

Kubernetes uses pluggable interfaces for flexibility:

CRI (Container Runtime Interface): Runs containers (e.g., containerd).

CNI (Container Network Interface): Sets up pod networking (e.g., Flannel).

CSI (Container Storage Interface): Manages storage (e.g., AWS EBS).

Example:

When a pod starts, the CRI tells containerd to create a container with specific resource limits using runc.


Watch API & Webhooks

Watch API: Monitors etcd for changes and notifies components (e.g., a controller recreates a deleted pod).
Webhooks:
Mutating: Modifies objects (e.g., injects sidecar containers).

Validating: Enforces policies (e.g., rejects pods without labels).



Real-World Example:

You accidentally delete a pod. The ReplicaSet controller, via the Watch API, notices and creates a new pod to maintain the desired replica count.


Key Directories

/etc/kubernetes/manifests: Holds static pod manifests (e.g., kube-apiserver, etcd).
/var/log: Stores logs for Kubernetes components and containers.

Check Enabled Admission Plugins:
cat /etc/kubernetes/manifests/kube-apiserver.yaml


Communication in K8s

Kubernetes uses Protobuf (Protocol Buffers) for fast, lightweight communication between components, outperforming JSON in speed and efficiency.
Example:

The kube-apiserver uses Protobuf to quickly read/write cluster state to etcd.


Get Hands-On!

Ready to try Kubernetes? Follow these steps:

Install Minikube (a local K8s cluster):minikube start


Play with kubectl:kubectl get nodes
kubectl get pods


Deploy a Sample App:kubectl create deployment my-app --image=nginx --replicas=3
kubectl expose deployment my-app --port=80 --type=ClusterIP




🎯 Challenge: Scale your deployment to 5 replicas with kubectl scale!


Contributing
Love this guide? Want to make it better? 🙌

Happy Kuberneting! 🚢 Let’s build scalable, resilient apps together!


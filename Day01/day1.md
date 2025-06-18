🏛️ Kubernetes Architecture Overview

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. The architecture is primarily divided into two sections: the Control Plane and the Worker Nodes, which work in tandem to maintain the desired state of your applications.



🌐 Key Components

🔹 Control Plane

Responsible for managing the entire Kubernetes cluster. It includes:

kube-apiserver: Central management component

etcd: Cluster state database

kube-scheduler: Assigns pods to nodes

kube-controller-manager: Runs various controllers


🔹 Worker Nodes

Run your application workloads. Key components:

kubelet: Node agent

kube-proxy: Handles networking

Container Runtime: Executes containers (e.g., containerd, CRI-O)


🔹 Add-ons

Extend cluster capabilities:

CNI (Container Network Interface): Networking

CSI (Container Storage Interface): Storage

Monitoring & Logging: e.g., Prometheus, Fluentd





🧩 Kube-API Server: The Gateway to Kubernetes

Acts as the front-end for the control plane and exposes the Kubernetes API.

📌 Key Responsibilities

Authentication: Verifies identity via certificates/tokens

Authorization: Checks if the authenticated user has permission

Admission Control: Validates or mutates incoming requests


🔄 Admission Controller Types

Mutating: Adds/updates default fields (e.g., memory limits)

Validating: Ensures requests conform to defined policies


💡 Real-World Scenario: Enforcing Resource Limits

Policy: All pods must have a memory limit of 500Mi Request: A pod is submitted with 1Gi memory Workflow:

1. Mutating webhook rewrites 1Gi to 500Mi


2. Validating webhook ensures policy compliance


3. Approved request is stored in etcd



Outcome: Prevents over-provisioning and enforces consistency




🗄️ etcd: The Heart of Kubernetes

A distributed, consistent key-value store that holds the entire cluster state.

🔑 Key Features

Distributed, NoSQL, highly available

Uses Raft consensus algorithm

Stores cluster specs, secrets, config, etc.

Write-Ahead Logging (WAL) for data durability


🛠️ How It Works

Write Flow: Log entry → Replication → Commit after quorum

Read Flow: Served by current leader node → Returned via API server


📌 Example

If a node fails, etcd retains the desired state. The controller checks etcd and re-creates pods on healthy nodes.




🔄 Controllers: Maintaining Desired State

Continuously monitor objects and ensure actual state matches desired state.

🛠️ Types of Controllers

Node Controller: Handles node availability

Service Controller: Manages service endpoints

ReplicaSet Controller: Maintains correct pod count

CronJob Controller: Schedules recurring jobs

Custom Controllers: Written using Operators


🚀 Example: ReplicaSet Controller

1. Deployment requests 5 replicas


2. Due to crash, only 2 running


3. Controller detects 3 missing


4. Automatically creates 3 new pods






⚖️ Scheduler: Placing Pods on Nodes

🔄 Scheduling Flow

1. Queue: Unscheduled pods queued


2. Filtering: Nodes checked for resource/constraints


3. Scoring: Eligible nodes ranked


4. Binding: Pod assigned to best node



🌍 Example

Pod needs: 2 vCPUs + GPU

2 nodes have GPU → Filter phase

Least-loaded GPU node picked → Score phase

Pod assigned → Bind phase


Customize via: Taints, Tolerations, Node Affinity, Pod Affinity




🧱 Kubelet: The Node Agent

Runs on each node and manages container lifecycle.

📌 Responsibilities

Communicates with kube-apiserver

Runs containers via container runtime

Watches /etc/kubernetes/manifests for static pods


🧪 Example

If a container crashes, kubelet restarts it per the pod's restartPolicy



🌐 Kube-Proxy: Networking Maestro

Handles network routing and traffic forwarding.

Core Duties

Maintains iptables/IPVS rules

Routes traffic to proper pod endpoints


📌 Example

Frontend pod calls backend via ClusterIP. Kube-proxy forwards the request to one of the backend pod IPs.




🐋 Container & Platform Interfaces

Kubernetes relies on standard interfaces:

Interface	Full Form	Role

CRI	Container Runtime Interface	Manages container lifecycle
CSI	Container Storage Interface	Manages persistent volumes
CNI	Container Network Interface	Manages pod networking


🔍 CRI Deep Dive

Uses containerd → talks to runc

Applies namespaces, cgroups, etc.

Example: runc enforces memory limits during container creation





📡 Watch API: Real-Time Updates

Watches for changes in etcd and notifies components.

🧠 How It Works

Controllers "watch" objects

On create/update/delete → get notified

Automatically reconcile the state


🌍 Example

User deletes a pod → Watch API → ReplicaSet controller notified → Creates replacement pod




🔗 Webhooks: Extending Kubernetes

Inject custom logic into admission control.

Types

Mutating Webhook: e.g., Add sidecar containers

Validating Webhook: e.g., Enforce required labels




📁 Important Kubernetes Directories

Directory	Purpose

/etc/kubernetes/manifests	Stores static pods (API server, etcd, etc.)
/var/log	Holds logs from Kubernetes components and containers


🔍 View Admission Plugins

cat /etc/kubernetes/manifests/kube-apiserver.yaml



🌐 Kubernetes Communication Protocol

Kubernetes uses Protobuf for fast, efficient communication.

Why Protobuf?

Faster and lighter than JSON

Reduces network & CPU load


📌 Example

Kube-apiserver → etcd: Uses Protobuf to read/write state quickly




✅ This architecture enables Kubernetes to deliver powerful automation, resiliency, and scalability for containerized workloads.




📘 Kubernetes Day 3

Kubernetes: Labels, Selectors, ReplicaSets, Deployments, and Deployment Strategies
A production-grade guide to mastering Kubernetes Labels, Selectors, ReplicaSets, Deployments, and Deployment Strategies. This repository provides in-depth explanations, practical examples, and best practices for running scalable and reliable applications in Kubernetes.


Labels
What Are Labels?
Labels are key-value pairs attached to Kubernetes objects (e.g., Pods, Services, Deployments) to organize and manage resources. They enable flexible grouping and selection for operations like scaling, routing, or monitoring.

Example:

metadata:
  labels:
    app: webserver
    env: production
    tier: frontend
    version: v1.0

Use Cases for Labels

Resource Organization: Group resources by application, environment, or team (e.g., team: devops).
Service Routing: Services use labels to route traffic to matching Pods.
Scaling and Updates: Deployments and ReplicaSets use labels to manage Pod replicas.
Monitoring and Logging: Tools like Prometheus use labels to filter metrics.

Best Practices for Labels

Use standardized keys (e.g., app.kubernetes.io/name, app.kubernetes.io/version) as per Kubernetes recommended labels.
Keep labels specific and minimal to avoid confusion.
Include hierarchical labels like env, tier, or region (e.g., region: us-east).
Automate labeling with tools like Helm or Kustomize.
Avoid storing sensitive data in labels.

Selectors

Types of Selectors

Selectors filter Kubernetes objects based on labels, enabling precise resource targeting.

Equality-Based Selectors:

Match exact key-value pairs (e.g., app=webserver, env!=staging).


Set-Based Selectors:

Use operators like In, NotIn, and Exists (e.g., env in (production, staging)).



Equality-Based Example:

selector:
  matchLabels:
    app: webserver
    tier: frontend



Set-Based Example:
selector:
  matchExpressions:
    - {key: env, operator: In, values: [production, staging]}
    - {key: tier, operator: Exists}



Advanced Selector Usage



Use set-based selectors for Services targeting Pods across multiple environments.
Combine selectors with kubectl for ad-hoc queries (e.g., kubectl get pods -l 'app=webserver,env=production').
Apply node selectors to schedule Pods on specific nodes (e.g., nodeSelector: {disktype: ssd}).



ReplicaSets
What Is a ReplicaSet?
A ReplicaSet ensures a specified number of Pod replicas are running, providing high availability and fault tolerance. It is typically managed by Deployments but can be used standalone for simple scenarios.
Production Considerations for ReplicaSets



Prefer Deployments over standalone ReplicaSets for declarative updates and rollbacks.
Use Horizontal Pod Autoscalers (HPA) to dynamically adjust replicas based on load.
Implement Pod Disruption Budgets (PDB) to limit disruptions during updates or maintenance.

ReplicaSet Example:


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver-rs
  labels:
    app: webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
      version: v1.0
  template:
    metadata:
      labels:
        app: webserver
        version: v1.0
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"





Deployments
What Is a Deployment?
A Deployment manages ReplicaSets and Pods, providing declarative updates, rollbacks, and scaling. It is the primary resource for running stateless applications in Kubernetes.
Production Features of Deployments




Rolling Updates: Gradually update Pods with zero downtime.
Rollback: Revert to previous versions using kubectl rollout undo.
Revision History: Track changes with kubectl rollout history.
Horizontal Scaling: Scale Pods manually or via HPA.
Self-Healing: Automatically replace failed Pods.

Deployment Example:



apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver-deployment
  labels:
    app: webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
      version: v1.0
  template:
    metadata:
      labels:
        app: webserver
        version: v1.0
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"

Deployment Strategies
Rolling Update
The default strategy, incrementally replacing old Pods with new ones to ensure zero downtime.
Configuration:
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%





Production Tips:



Adjust maxSurge and maxUnavailable to balance speed and availability.
Use readiness probes to ensure new Pods are healthy before serving traffic.
Monitor with kubectl rollout status deployment/webserver.



Recreate
Terminates all old Pods before creating new ones, causing downtime.
Configuration:
spec:
  strategy:
    type: Recreate

Use Case: Applications requiring exclusive version runs (e.g., database migrations).
Blue/Green Deployment

Run two versions (blue and green) side-by-side, then switch traffic by updating the Service selector.
Steps:




Deploy the new version with a different label (e.g., version: v2.0).
Update the Service to point to the new version.
Delete the old version after verification.



Service Example:



apiVersion: v1
kind: Service
metadata:
  name: webserver-service
spec:
  selector:
    app: webserver
    version: v2.0
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80



Canary Deployment
Route a small percentage of traffic to the new version for testing.
Manual Steps:



Deploy v2 with fewer replicas and a unique label (e.g., version: v2.0).
Configure the Service to include both v1 and v2 Pods.
Gradually scale v2 replicas and reduce v1 replicas.



Advanced: Use ingress controllers (e.g., NGINX, Istio) for precise traffic splitting.
A/B Testing
Test features by directing user segments to different versions based on headers or cookies. Requires a service mesh (e.g., Istio).



Istio Example:




apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver-vs
spec:
  hosts:
  - webserver
  http:
  - match:
    - headers:
        user-type:
          exact: beta
    route:
    - destination:
        host: webserver
        subset: v2
      weight: 100
    - destination:
        host: webserver
        subset: v1
      weight: 100

Production Best Practices

Health Checks:
Use readinessProbe and livenessProbe for Pod reliability.


Resource Management:
Define requests and limits to prevent resource starvation.
Example: requests: {cpu: "100m", memory: "128Mi"}.


Pod Disruption Budgets:
Ensure availability during disruptions.
Example:apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webserver-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: webserver




Label Consistency: Standardize labels across resources.
Monitoring: Integrate with Prometheus, Grafana, or ELK.
CI/CD: Use ArgoCD or Flux for GitOps-driven deployments.
Rollback Planning: Test rollbacks in staging and use kubectl rollout undo.

Troubleshooting Tips


Pods Not Starting: Check kubectl describe pod <pod-name> for image or resource issues.
Service Not Routing: Verify selector labels with kubectl get pods -l app=webserver.
Rollout Stuck: Inspect kubectl rollout status deployment/webserver.
High Resource Usage: Monitor with kubectl top.
Selector Issues: Validate with kubectl get <resource> -l <selector>.

Resources


Kubernetes Documentation
Kubernetes Recommended Labels
Istio Traffic Management
Horizontal Pod Autoscaler
Kubernetes Examples


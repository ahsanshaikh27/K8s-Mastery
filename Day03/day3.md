📘 Kubernetes Day 3 —

# Kubernetes Production Pods Setup

This repository demonstrates best practices for running Kubernetes Pods in production environments with a focus on:

* Pod Lifecycle & Health
* Sidecar vs Init Containers
* Security using Trivy and Kyverno
* Pod Restart Policies & Termination
* Production-grade YAML examples

---

## 📦 1. Pods in Production

### Key Considerations:

* Always deploy Pods using higher-level controllers like **Deployment** (for stateless apps) or **StatefulSet** (for stateful apps).
* Define **liveness** and **readiness probes** to monitor pod health and ensure traffic is only routed to healthy containers.
* Configure **resource requests and limits** to enable efficient resource scheduling and prevent pod eviction due to resource overuse.
* Set the appropriate `imagePullPolicy`:

  * `Always`: Ensures the latest image is pulled.
  * `IfNotPresent`: Uses the locally cached image if available.

### Pod Lifecycle Phases:

1. **Pending**: The Pod has been accepted but one or more containers are not yet created.
2. **Running**: The Pod has been scheduled and all containers are running.
3. **Succeeded**: All containers have terminated successfully.
4. **Failed**: At least one container terminated with an error.
5. **Unknown**: The state of the Pod could not be determined.

---

## 🧩 2. Sidecar vs Init Containers

Both Init and Sidecar containers are auxiliary containers used alongside the main application container, but they serve different purposes:

| Feature              | Init Container                         | Sidecar Container                         |
| -------------------- | -------------------------------------- | ----------------------------------------- |
| **Purpose**          | Prepares environment before app starts | Enhances functionality alongside main app |
| **Lifecycle**        | Runs sequentially before app container | Runs in parallel with app container       |
| **Common Use Cases** | Database migrations, config downloads  | Logging, monitoring, reverse proxies      |
| **Restart Behavior** | Restarts until successful              | Follows Pod restart policy                |

### Example Use Cases:

* **Init Container**: Download required configuration files, run database migrations, wait for a dependent service to be available.
* **Sidecar Container**: Stream logs to external service, collect metrics, or provide a proxy like Envoy.

---

## 🛡️ 3. Security Tools

### ✅ Trivy - Vulnerability Scanner

Trivy is a simple and comprehensive tool to scan container images for vulnerabilities.

```bash
trivy image nginx:latest
```

* Scans OS-level packages and app dependencies.
* Identifies known CVEs.
* Can be automated within CI/CD pipelines.

### ✅ Kyverno - Kubernetes Policy Engine

Kyverno helps enforce security and best practices at the cluster level.

#### Use Kyverno for:

* **Validation**: Prevent usage of `latest` tag or privileged containers.
* **Mutation**: Automatically label pods or inject configurations.
* **Enforcement**: Ensure containers follow policy guidelines.

#### Example: Block Pods using `latest` tag

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
spec:
  validationFailureAction: enforce
  rules:
    - name: validate-image-tag
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "Using 'latest' tag is not allowed"
        pattern:
          spec:
            containers:
              - image: "!*:latest"
```

---

## 🔁 4. Restart Policies

Pod restart behavior is controlled by the `restartPolicy` field:

| Policy      | Description                                         |
| ----------- | --------------------------------------------------- |
| `Always`    | Always restarts the container if it exits (default) |
| `OnFailure` | Restarts container only on non-zero exit codes      |
| `Never`     | Container is not restarted after it exits           |

### ⚠️ Backoff Strategy:

* Kubernetes uses an **exponential backoff** strategy:

  * 10s → 20s → 40s ... up to 5 minutes.
* Useful in preventing constant retries that consume resources.

---

## ⛔️ 5. Pod Termination

When deleting a Pod, Kubernetes follows this termination sequence:

### Termination Flow:

1. `kubectl delete pod` is issued.
2. Pod enters **Terminating** state.
3. A **grace period** (default: 30 seconds) begins.
4. Kubernetes sends a **SIGTERM** signal to containers.
5. After grace period, **SIGKILL** is sent to forcibly terminate the container.

### Force Delete:

To skip the grace period:

```bash
kubectl delete pod <pod-name> --grace-period=0 --force
```

---

## 🔐 6. Security Context

Security contexts define privilege and access controls for pods or containers.

### Best Practices:

* Run as a non-root user.
* Disable privilege escalation.
* Avoid running privileged containers.

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
```

---

## 🛠️ 7. Tools to Use

| Tool        | Purpose                              |
| ----------- | ------------------------------------ |
| Trivy       | Image vulnerability scanning         |
| Kyverno     | Kubernetes policy enforcement        |
| K9s         | TUI for managing Kubernetes clusters |
| Stern       | Tail logs from multiple pods         |
| Kube-linter | YAML configuration linting           |

---

## 🧪 8. Production YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      securityContext:
        runAsNonRoot: true
      initContainers:
        - name: init-migrate
          image: myapp/db-migrate:1.0
          command: ["sh", "-c", "python manage.py migrate"]
      containers:
        - name: main-app
          image: myapp/web:2.0
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "500m"
              memory: "1Gi"
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /ready
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
        - name: sidecar-logger
          image: fluent/fluentd
      imagePullSecrets:
        - name: regcred
      restartPolicy: Always
```

---

## 📚 Resources

* [Kyverno Documentation](https://kyverno.io)
* [Trivy Documentation](https://aquasecurity.github.io/trivy/)
* [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
* [Security Context in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

---

## ✅ Summary

* Avoid `:latest` tags and scan images regularly.
* Enforce security and compliance using Kyverno.
* Use Init Containers for setup and Sidecars for support services.
* Always define health probes and resource limits.
* Understand and configure restart policies and termination behavior.

> Focus on building stable, secure, and observable Kubernetes workloads for production environments.


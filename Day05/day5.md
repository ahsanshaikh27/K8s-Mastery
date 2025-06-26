---

# Day 5: Adding SSL/TLS to Your Kubernetes Application with Cert-Manager and Gateway API

## 🎯 Objective

Secure Kubernetes-based application with **HTTPS** using **Cert-Manager** for automated TLS certificate management, the **Gateway API** for modern traffic routing, and **Namespaces** for logical isolation. This guide leverages **Let's Encrypt** for free, auto-renewing certificates to ensure secure communication.

---

## 🧠 Key Concepts

### 🔐 What is TLS?
- **Transport Layer Security (TLS)** is a cryptographic protocol that ensures secure communication over a network by providing:
  - **Confidentiality**: Encrypts data to prevent eavesdropping.
  - **Integrity**: Ensures data is not tampered with during transit.
  - **Authenticity**: Verifies the identity of the server (and optionally the client).
- TLS powers `https://` and is the modern successor to SSL.

### 📦 What is Cert-Manager?
- **Cert-Manager** is a Kubernetes-native controller that automates the issuance, renewal, and management of TLS certificates.
- Supports certificate authorities (CAs) like **Let's Encrypt**, HashiCorp Vault, and private CAs.
- Simplifies certificate lifecycle management, reducing operational overhead.

### 🌐 What is the Gateway API?
- The **Gateway API** is a modern, extensible Kubernetes API for managing Layer 4 (L4) and Layer 7 (L7) traffic routing.
- Replaces the older Ingress API with more flexibility and features, including native TLS support.
- Enables fine-grained control over HTTP/HTTPS routing and integrates seamlessly with Cert-Manager.

### 🗂 Why Use Namespaces?
- **Namespaces** provide logical isolation within a Kubernetes cluster.
- Enable separation of environments (e.g., dev, staging, prod) or teams.
- Enhance security by restricting access and scoping resources like certificates and secrets.

---

## 🧰 Use Cases

| Scenario | Why It Matters |
|----------|---------------|
| 🔒 **HTTPS Web Application** | Ensures secure communication between users and backend services, protecting sensitive data. |
| 🔄 **Auto-Renewing Certificates** | Eliminates manual certificate renewals, preventing downtime due to expired certificates. |
| 🔐 **Mutual TLS (mTLS)** | Extends security for service-to-service communication within the cluster. |
| 🧪 **Staging vs. Production** | Manages different certificates and domains across isolated namespaces for testing and production. |

---

## 🛠 Prerequisites

Before proceeding, ensure you have:
- A running **Kubernetes cluster** (e.g., Minikube, EKS, GKE, or AKS).
- **Helm** installed (version `3.x` or higher).
- A **Gateway controller** installed (e.g., NGINX, Istio, or Kubernetes Gateway).
- A **domain name** pointing to your cluster’s ingress IP (e.g., `myapp.example.com`).
- Access to modify **DNS records** for the domain.
- `kubectl` configured to interact with your cluster.

---

## 🏗 Implementation Steps

Follow these steps to secure your application with TLS using Cert-Manager and Gateway API.

### ✅ Step 1: Create a Namespace
Namespaces provide isolation for your application resources.

```bash
kubectl create namespace web-secure
```

### ✅ Step 2: Install Cert-Manager
Cert-Manager automates certificate issuance and renewal. Install it using Helm.

```bash
# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install Cert-Manager with CRDs
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

Verify the installation:
```bash
kubectl get pods -n cert-manager
```

Ensure all Cert-Manager pods are in the `Running` state.

### ✅ Step 3: Configure a ClusterIssuer for Let's Encrypt
A `ClusterIssuer` defines how Cert-Manager interacts with Let's Encrypt to issue certificates.

Create a file named `cluster-issuer.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com # Replace with your email
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx # Replace with your ingress controller class
```

Apply the configuration:
```bash
kubectl apply -f cluster-issuer.yaml
```

> **Note**: Replace `nginx` with the appropriate ingress class for your Gateway controller (e.g., `istio`, `traefik`).

### ✅ Step 4: Request a TLS Certificate
Define a `Certificate` resource to request a TLS certificate for your domain.

Create a file named `certificate.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-app-tls
  namespace: web-secure
spec:
  secretName: my-app-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - myapp.example.com # Replace with your domain
```

Apply the configuration:
```bash
kubectl apply -f certificate.yaml
```

Cert-Manager will:
1. Request a certificate from Let's Encrypt.
2. Store the certificate as a Kubernetes secret named `my-app-tls-secret` in the `web-secure` namespace.

### ✅ Step 5: Configure the Gateway API with TLS
Define a `Gateway` resource to handle HTTPS traffic using the certificate.

Create a file named `gateway.yaml`:

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: my-app-gateway
  namespace: web-secure
spec:
  gatewayClassName: nginx # Replace with your Gateway controller class
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      certificateRefs:
      - kind: Secret
        name: my-app-tls-secret
    allowedRoutes:
      namespaces:
        from: Same
```

Apply the configuration:
```bash
kubectl apply -f gateway.yaml
```

> **Note**: Ensure the `gatewayClassName` matches your installed Gateway controller.

### ✅ Step 6: Define Routing with HTTPRoute
Create an `HTTPRoute` to route traffic to your application service.

Create a file named `httproute.yaml`:

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: web-secure
spec:
  parentRefs:
  - name: my-app-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: my-app-service # Replace with your service name
      port: 80
```

Apply the configuration:
```bash
kubectl apply -f httproute.yaml
```

> **Note**: Replace `my-app-service` with the name of your application’s Kubernetes service.

### 🔎 Step 7: Validate and Debug
Verify the certificate and Gateway setup:
```bash
# Check certificate status
kubectl describe certificate my-app-tls -n web-secure

# View events for troubleshooting
kubectl get events -n web-secure

# Inspect the TLS secret
kubectl get secret my-app-tls-secret -n web-secure -o yaml
```

Test your application by accessing `https://myapp.example.com` in a browser or using `curl`.

---

## 🛡 Best Practices

1. **Use Staging Let's Encrypt Issuer**: Test with Let's Encrypt's staging server (`https://acme-staging-v02.api.letsencrypt.org/directory`) to avoid rate limits during development.
2. **Automate DNS Validation**: For wildcard certificates or complex setups, use DNS-01 solvers (e.g., with Cloudflare or Route 53).
3. **Separate Issuers for Environments**: Use distinct `ClusterIssuer` resources for staging and production to avoid conflicts.
4. **Monitor Certificate Expiry**: Set up alerts for certificate expiration, even with auto-renewal, to catch potential issues.
5. **Restrict Namespace Access**: Use RBAC to limit access to the `web-secure` namespace for enhanced security.
6. **Backup Secrets**: Regularly back up the `my-app-tls-secret` secret to prevent data loss.

---

## 📁 Folder Structure

```bash
Day5-SSL-Cert/
├── cluster-issuer.yaml   # Configures Let's Encrypt ClusterIssuer
├── certificate.yaml      # Defines the TLS certificate
├── gateway.yaml          # Configures the HTTPS Gateway
├── httproute.yaml        # Routes traffic to the application
└── README.md             # This documentation
```

---

## 📌 Summary

By completing this setup:
- Secured your application with **HTTPS** using TLS certificates.
- Automated certificate issuance and renewal with **Cert-Manager**.
- Configured modern traffic routing with the **Gateway API**.
- Organized resources using **Namespaces** for isolation.

 Application is now production-ready with industry-standard encryption and minimal maintenance overhead.

---
## 🔗 Resources

- [Cert-Manager Documentation](https://cert-manager.io/docs/)
- [Gateway API Specification](https://gateway-api.sigs.k8s.io/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Kubernetes Gateway API Guide](https://kubernetes.io/docs/concepts/services-networking/gateway/)

---



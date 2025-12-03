# **Migration from NGINX Ingress Controller → Kubernetes Gateway API (kubeadm Cluster on AWS EC2)**

This repository demonstrates the **complete migration** from **NGINX Ingress** to the modern **Kubernetes Gateway API**, including **real-world errors**, **fixes**, **YAMLs**, and **Mermaid.js architecture diagrams**.

---

# 🚀 Overview

The lab includes:

* Deploying Ingress Controller
* Deploying Gateway API
* Migrating routing
* Handling all real-world issues
* Testing with curl using Host headers
* Exposing gateway controller
* Full architecture diagrams

---

# 📂 Repository Structure

```
k8s-nginx-to-gateway-api-migration/
├── manifests/
├── diagrams/
└── logs/
```

---

# 🧱 **1. Ingress Architecture**

```mermaid
flowchart LR
    Client --> NodePort[NodePort Service 31050]
    NodePort --> IngressCtrl[NGINX Ingress Controller Pod]
    IngressCtrl --> IngressRule[Ingress Rule]
    IngressRule --> Service[ClusterIP Service]
    Service --> Pods[Application Pods]
```

---

# 🧱 **2. Gateway API Architecture**

```mermaid
flowchart LR
    Client --> GatewayNodePort[NodePort Service 32080]
    GatewayNodePort --> NGWC[NGINX Gateway Controller Pod]
    NGWC --> Gateway[Gateway Listener 80]
    Gateway --> HTTPRoute[HTTPRoute Match Rules]
    HTTPRoute --> Service[Backend Service]
    Service --> Pods[Application Pods]
```

---

# 🔁 **3. Migration Workflow — Ingress → Gateway API**

```mermaid
sequenceDiagram
    participant User
    participant Ingress
    participant GatewayAPI
    participant App

    User->>Ingress: Send HTTP Request (Host: myapp.local)
    Ingress->>App: Route traffic
    Note over User,Ingress: Phase 1: Ingress Working

    User->>GatewayAPI: Apply Gateway + HTTPRoute
    GatewayAPI->>App: Route Traffic (Parallel)
    Note over GatewayAPI,App: Phase 2: Ingress & Gateway both active

    User->>Ingress: Remove Ingress Rule
    Ingress-->>User: 404 (Removed)
    User->>GatewayAPI: Verify Gateway Traffic
    GatewayAPI->>App: Handles traffic fully
    Note over GatewayAPI,App: Migration Complete
```

---

# 🌐 **4. Difference Between Ingress & Gateway API**

```mermaid
classDiagram
    class Ingress {
        +Host based routing
        +Path based routing
        -Limited extensibility
        -Controller specific annotations
    }

    class GatewayAPI {
        +GatewayClass
        +Listeners
        +HTTPRoute / TCPRoute / GRPCRoute
        +Traffic splitting
        +Header methods, filters, rewrites
        +Multi-tenant support
    }

Ingress <|-- GatewayAPI
```

---

# 🔍 **5. Request Flow Comparison**

## Ingress Request Flow

```mermaid
sequenceDiagram
    Client->>NodePort: HTTP request
    NodePort->>IngressCtrl: Forward traffic
    IngressCtrl->>IngressRule: Host/Path match
    IngressRule->>Service: Send request
    Service->>Pods: Return response
```

## Gateway API Request Flow

```mermaid
sequenceDiagram
    Client->>GatewayNodePort: HTTP request
    GatewayNodePort->>GatewayCtrl: Forward traffic
    GatewayCtrl->>Gateway: Listener filter
    Gateway->>HTTPRoute: Match Host + Path
    HTTPRoute->>Service: Forward request
    Service->>Pods: Return response
```

---

# 🧩 **6. Real Errors & Fixes**

## 🔥 Error 1 — Ingress Admission Webhook Timeout

```
failed calling webhook "validate.nginx.ingress.kubernetes.io"
context deadline exceeded
```

### Fix

```
kubectl delete validatingwebhookconfiguration ingress-nginx-admission
```

---

## 🔥 Error 2 — Gateway Controller YAML 404

Cause: Wrong URL — upstream changed folder structure.

### Fix

```
https://raw.githubusercontent.com/nginxinc/nginx-kubernetes-gateway/v1.3.0/deploy/manifests/nginx-gateway.yaml
```

---

## 🔥 Error 3 — GatewayClass is Immutable

Cause: GatewayClass auto-created by controller.

### Fix

Do not create manually — use existing one:

```
kubectl get gatewayclass
```

---

## 🔥 Error 4 — Gateway API not responding

No Service exists in `nginx-gateway` namespace.

### Fix

Create NodePort service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: nginx-gateway
  ports:
    - port: 80
      nodePort: 32080
```

---

# 🛠 **7. Full Workflow Summary Diagram**

```mermaid
flowchart TD
    A[Deploy App + Service] --> B[Install Ingress Controller]
    B --> C[Create Ingress Rule]
    C --> D[Test Ingress via NodePort]
    D --> E[Install Gateway API CRDs]
    E --> F[Install NGINX Gateway Controller]
    F --> G[Create Gateway]
    G --> H[Create HTTPRoute]
    H --> I[Expose Gateway Controller Service]
    I --> J[Test Gateway API via NodePort]
    J --> K[Remove Ingress]
    K --> L[Gateway API Fully Active]
```

---

# ✔ Final Architecture After Migration

```mermaid
flowchart LR
    subgraph Node
        GatewayNodePort[NodePort 32080]
        NginxGWC[NGINX Gateway Controller]
    end

    Client --> GatewayNodePort
    GatewayNodePort --> NginxGWC
    NginxGWC --> Gateway
    Gateway --> HTTPRoute
    HTTPRoute --> K8sService
    K8sService --> Pods
```

---

# 🎉 **Congratulations**

You now have a **complete, production-grade migration example**, including:

✔ Ingress → Gateway API migration
✔ All manifests
✔ Troubleshooting
✔ Architecture diagrams
✔ Repository structure
✔ Final working environment


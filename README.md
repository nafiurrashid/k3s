# k3s Learning Journey

This repository documents my hands-on journey learning Kubernetes using K3s, starting from Pods and progressing through Deployments, Services, DNS, Ingress, RBAC, Observability, and Production Operations.

The goal is to understand not just how to use Kubernetes, but how it works internally.

---

## Learning Roadmap

### Phase 1 — Core Kubernetes Objects

| Exercise                                    | Topic                                  |
| ------------------------------------------- | -------------------------------------- |
| [1.1 - Pods](./1.1.md)                      | Creating and managing standalone Pods  |
| [1.2 - Deployments & ReplicaSets](./1.2.md) | Self-healing, scaling, rolling updates |
| [1.3 - Services & DNS](./1.3.md)            | Service discovery, ClusterIP, NodePort |

---

## Supporting Resources

### Interactive Learning Tool

* [kubectl apply Interactive Explainer](./kubectl_apply_interactive_explainer.html)

A visual walkthrough of how `kubectl apply` works and how Kubernetes reconciles desired state.

---

## Kubernetes Architecture Progression

```text
Pod
 ↓
Deployment
 ↓
ReplicaSet
 ↓
Service
 ↓
DNS
 ↓
Ingress
 ↓
Load Balancer
```

---

## Exercises Completed

### 1.1 — Pods

Learned:

* Pod lifecycle
* Resource requests and limits
* Liveness probes
* Readiness probes
* Port forwarding
* Pod inspection and debugging

File:

```text
1.1.md
```

---

### 1.2 — Deployments & ReplicaSets

Learned:

* Deployments
* ReplicaSets
* Desired state
* Self-healing
* Scaling
* Rolling updates
* Rollbacks

File:

```text
1.2.md
```

---

### 1.3 — Services & DNS

Learned:

* ClusterIP Services
* NodePort Services
* Service selectors
* Endpoints
* Service discovery
* Kubernetes DNS
* kube-proxy basics

File:

```text
1.3.md
```

---

## Upcoming Topics

* Ingress Controllers
* Traefik
* NGINX Ingress
* TLS Certificates
* ConfigMaps
* Secrets
* RBAC
* Service Accounts
* Persistent Volumes
* Storage Classes
* Prometheus
* Grafana
* Loki
* OpenTelemetry
* Horizontal Pod Autoscaling
* High Availability K3s
* EKS

---

## Lab Environment

Current environment:

```text
K3s
 ├─ Single-node cluster
 └─ AWS EC2
```

Future environment:

```text
K3s HA Cluster
 ├─ EC2-1
 ├─ EC2-2
 └─ EC2-3
```

---

## Repository Structure

```text
.
├── README.md
├── 1.1.md
├── 1.2.md
├── 1.3.md
└── kubectl_apply_interactive_explainer.html
```

---

## Learning Philosophy

The objective is to understand:

* How Kubernetes schedules workloads
* How networking works
* How services discover each other
* How high availability is achieved
* How production clusters are operated
* How managed Kubernetes (EKS) differs from self-managed Kubernetes (K3s)

This repository serves as both a learning journal and a future reference guide.


# K3s vs Kubernetes (K8s)

## The Core Idea

K3s and Kubernetes run the same Kubernetes API and reconciliation logic.

Every Kubernetes manifest you write works on both.

The difference is mainly:

- Packaging
- Default components
- Operational responsibility

K3s is not a different Kubernetes. It is a lightweight Kubernetes distribution.

---

# 1. The Binary

## Kubernetes (K8s)

Kubernetes ships as separate components:

- `kube-apiserver`
- `kube-scheduler`
- `kube-controller-manager`
- `kube-proxy`
- `kubelet`

Each:

- Runs as its own process
- Has its own configuration
- Can fail independently

On a kubeadm cluster, control plane components usually run as static Pods managed by kubelet.

Check:

```bash
kubectl get pods -n kube-system
```

You will see components like:

```text
kube-apiserver
kube-controller-manager
kube-scheduler
```

---

## K3s

K3s packages Kubernetes components into a single lightweight binary.

Example:

```text
k3s
```

Managed by:

```text
k3s.service
```

Modes:

```text
server → control plane
agent  → worker node
```

K3s removes unnecessary components and simplifies packaging:

- Removes legacy features
- Removes alpha APIs
- Removes in-tree cloud provider integrations

This reduces:

- Binary size
- Memory usage
- Operational complexity

---

## Practical Difference

K3s:

```bash
sudo systemctl restart k3s
```

Kubernetes:

You manage individual components or static pod definitions.

---

# 2. The Datastore

This is the biggest architectural difference.

---

# Kubernetes (K8s)

Kubernetes requires:

```text
etcd
```

etcd is a distributed key-value database.

It stores:

- Pods
- Deployments
- Services
- Secrets
- Configurations
- Cluster state

Typical HA setup:

```text
3 or 5 etcd members
```

Example:

```text
etcd-1
etcd-2
etcd-3
```

---

## etcd Operations

You are responsible for:

- Backups
- Snapshots
- Member replacement
- Upgrades
- Disaster recovery

If etcd loses quorum:

```text
Cluster cannot operate normally
```

---

# K3s

Default datastore:

```text
SQLite
```

Location:

```text
/var/lib/rancher/k3s/server/db/state.db
```

Advantages:

- No separate database process
- No quorum management
- Very simple setup
- Fast startup

Tradeoff:

A single-server K3s cluster is not highly available.

---

# K3s HA Datastore Options

## Embedded etcd

Example:

```bash
--cluster-init
```

Used for:

```text
Multi-server HA K3s
```

---

## External Database

K3s can use:

- MySQL
- PostgreSQL

using:

```bash
--datastore-endpoint
```

---

Important:

Kubernetes itself requires etcd.

It does not support MySQL/PostgreSQL as the API server datastore.

---

# 3. Networking (CNI)

CNI = Container Network Interface

It provides networking between Pods.

---

# K3s

K3s includes:

```text
Flannel
```

by default.

Flannel provides:

```text
VXLAN overlay networking
```

Advantages:

- Simple
- Works almost everywhere

Limitation:

Flannel does not enforce Kubernetes NetworkPolicy.

Example:

```yaml
kind: NetworkPolicy
```

The object exists, but enforcement requires a CNI that supports policies.

---

## NetworkPolicy-capable CNIs

Examples:

### Calico

Features:

- NetworkPolicy enforcement
- BGP routing
- IP-in-IP networking

---

### Cilium

Uses:

```text
eBPF
```

Features:

- Advanced networking
- L7 policies
- Better observability
- Modern dataplane

---

### kube-router

Another option for:

- Routing
- NetworkPolicy enforcement

---

# Kubernetes (kubeadm)

Kubernetes installs with:

```text
No CNI
```

After:

```bash
kubeadm init
```

nodes remain:

```text
NotReady
```

until a CNI is installed.

This gives operators freedom to choose:

- Calico
- Cilium
- Flannel
- Others

---

# 4. Ingress and Load Balancing

# K3s

K3s includes:

- Traefik Ingress Controller
- ServiceLB

by default.

Example:

```yaml
type: LoadBalancer
```

works immediately.

However:

ServiceLB is a lightweight solution.

It is not the same as:

- AWS NLB
- AWS ALB
- Cloud load balancers

---

# Kubernetes (K8s)

Kubernetes does not include:

- Ingress controller
- Cloud load balancer integration

Example:

```yaml
type: LoadBalancer
```

without a cloud integration:

```text
Pending
```

You install:

- AWS Load Balancer Controller
- MetalLB
- NGINX Ingress
- Traefik

yourself.

---

# 5. Storage

# K3s

K3s includes:

```text
local-path-provisioner
```

as the default StorageClass.

It creates volumes using local node storage.

Advantages:

- Works immediately
- No setup required

Limitation:

Volumes are node-local.

Example:

```text
Pod
 |
Volume
 |
Node A
```

If the Pod moves:

```text
Node A → Node B
```

the original data is not automatically available.

---

# Kubernetes (K8s)

Kubernetes ships with:

```text
No default StorageClass
```

You install CSI drivers.

Examples on AWS:

---

## EBS CSI Driver

Provides:

```text
Block Storage
```

Usually:

```text
ReadWriteOnce
```

---

## EFS CSI Driver

Provides:

```text
Shared Storage
```

Usually:

```text
ReadWriteMany
```

Benefits:

- Snapshots
- Expansion
- Better portability

Requires:

- IAM permissions
- CSI installation
- StorageClass setup

---

# 6. What K3s Removes

To reduce size and complexity, K3s removes:

- In-tree cloud provider plugins
- Alpha APIs
- Separate cloud-controller-manager process

These changes do not affect normal Kubernetes usage.

The following work the same:

- Pods
- Deployments
- Services
- Ingress
- ConfigMaps
- Secrets
- RBAC

---

# 7. Security Defaults

# K3s

Automatically creates:

- Certificate Authority
- Certificates

Uses:

```text
Cluster registration token
```

Default behavior:

- Secrets stored in datastore
- Encryption at rest not enabled by default
- Control plane runs as root

---

# Kubernetes (kubeadm)

Similar defaults.

More customization options:

## Encrypt Secrets

```text
--encryption-provider-config
```

## Enable API Audit Logs

```text
--audit-log-path
```

More flexibility, but more operational responsibility.

---

# 8. HA Comparison

| Aspect | K3s HA | Kubernetes HA |
|-|-|-|
| Control Plane Nodes | 3+ odd number | 3+ odd number |
| Datastore | Embedded etcd | Stacked/External etcd |
| Load Balancer | Simple built-in options | Usually required |
| Setup Complexity | Lower | Higher |
| Failure Tolerance | 1 node failure with 3 nodes | Same |
| etcd Management | Simplified | Operator responsibility |

---

# K3s HA Flow

First server:

```bash
--cluster-init
```

Additional servers:

```bash
K3S_URL=<server>
K3S_TOKEN=<token>
```

K3s manages:

- etcd membership
- cluster joining
- server configuration

---

# Kubernetes HA Flow

Requires:

- Control plane nodes
- etcd cluster
- API server load balancer
- kubeadm HA configuration

More flexibility.

More operational work.

---

# 9. Resource Footprint

K3s generally consumes fewer resources because:

- Components are packaged together
- Defaults are optimized
- Less overhead

Full Kubernetes typically requires more resources because:

- Components run separately
- etcd is external
- More flexibility exists

Exact numbers depend on:

- Version
- Workload
- Configuration

---

# Why K3s Exists

K3s is designed for:

- Edge computing
- IoT
- Small servers
- Development environments
- Lightweight production clusters

It can run on:

- Raspberry Pi
- Small VMs
- Edge devices

while remaining Kubernetes-compatible.

---

# 30-Second Mental Model

K3s is similar to the relationship:

```text
SQLite  → PostgreSQL
```

SQLite:

- Simple
- Lightweight
- Easy to operate

PostgreSQL:

- More powerful
- More operational responsibility

Similarly:

| K3s | Kubernetes |
|-|-|
| Lightweight | Full-featured |
| Simple defaults | More choices |
| Lower resource usage | Higher resource usage |
| Easier operations | More control |

---

# Key Takeaway

Moving from K3s to Kubernetes:

Your manifests do not change.

These remain the same:

```yaml
Deployment
Service
Pod
Ingress
RBAC
ConfigMap
Secret
```

What changes is the operational model:

- Datastore management
- Backup strategy
- Control plane recovery
- CNI selection
- Storage drivers
- Load balancing
- HA architecture

The Kubernetes API is the same.

The responsibility of operating the cluster is what changes.

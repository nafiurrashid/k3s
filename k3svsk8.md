````md
# K3s vs Kubernetes (K8s)

## The Core Idea

K3s and Kubernetes run the same Kubernetes API and reconciliation logic. Every manifest you write works on both.

The difference is entirely in:

- Packaging
- Defaults
- Operational responsibilities

---

## 1. The Binary

### Kubernetes (K8s)

Kubernetes ships as separate binaries:

- `kube-apiserver`
- `kube-scheduler`
- `kube-controller-manager`
- `kube-proxy`
- `kubelet`

Each:

- Runs as its own process
- Has its own configuration
- Can fail independently

On a kubeadm cluster, these typically run as static Pods managed by kubelet.

```bash
kubectl get pods -n kube-system
```

You'll see the control plane components listed individually.

### K3s

K3s compiles all core Kubernetes components into a single binary (~70 MB).

One binary, one service:

```bash
k3s.service
```

Modes:

- `server` → control plane
- `agent` → worker node

Rancher removed:

- Alpha APIs
- Legacy features
- In-tree cloud provider plugins

This significantly reduces size and complexity.

### Practical Difference

K3s:

```bash
sudo systemctl restart k3s
```

Kubernetes:

- Restart individual components
- Modify static pod manifests

---

## 2. The Datastore

This is the biggest architectural difference.

### Kubernetes (K8s)

Requires:

```text
etcd
```

A distributed key-value store.

Typical HA setup:

```text
3 or 5 etcd nodes
```

etcd stores:

- Pods
- Deployments
- Services
- Secrets
- Entire cluster state

If etcd is unavailable:

```text
API Server becomes unavailable
```

If etcd loses quorum:

```text
Cluster becomes unusable
```

### Operational Responsibilities

You manage:

- Backups
- Snapshots
- Member replacement
- Upgrades
- Recovery

---

### K3s

Default datastore:

```text
SQLite
```

Location:

```text
/var/lib/rancher/k3s/server/db/state.db
```

Benefits:

- No separate database process
- No quorum
- No etcd management
- Fast startup

Tradeoff:

```text
Single-server setup is not HA
```

---

### HA K3s Options

#### Embedded etcd

```bash
--cluster-init
```

Recommended for production HA.

#### External Database

```bash
--datastore-endpoint
```

Supported databases:

- MySQL
- PostgreSQL

This is unique to K3s.

Full Kubernetes cannot use MySQL/Postgres as its datastore.

---

## 3. Networking (CNI)

### K3s

Ships with:

```text
Flannel
```

automatically configured.

Uses:

```text
VXLAN overlay networking
```

Pros:

- Simple
- Works everywhere

Cons:

- Does NOT enforce NetworkPolicy

Example:

```yaml
kind: NetworkPolicy
```

The object is stored, but nothing enforces it.

To enforce policies:

- Install kube-router
- Replace Flannel with another CNI

---

### Kubernetes (K8s)

Ships with:

```text
No CNI
```

After:

```bash
kubeadm init
```

Nodes remain:

```text
NotReady
```

until a CNI is installed.

---

### Common CNI Choices

#### Calico

Most common production choice.

Features:

- NetworkPolicy enforcement
- BGP routing
- IP-in-IP support

---

#### Cilium

Modern choice.

Uses:

```text
eBPF
```

Benefits:

- Faster networking
- L7 policies
- Better observability

---

#### Weave

Simpler but generally slower.

---

## 4. Ingress and Load Balancing

### K3s

Ships with:

- Traefik
- ServiceLB (formerly klipper-lb)

enabled by default.

This means:

```yaml
type: LoadBalancer
```

works immediately.

---

### Kubernetes (K8s)

Ships with:

- No ingress controller
- No load balancer

Example:

```yaml
type: LoadBalancer
```

Result:

```text
Pending
```

until you install:

- AWS Load Balancer Controller
- MetalLB
- Other load-balancer solutions

---

## 5. Storage

### K3s

Ships with:

```text
local-path-provisioner
```

Default StorageClass.

Creates directories on local node storage.

Benefits:

- Works immediately
- No configuration

Limitation:

```text
Volumes are tied to a single node
```

If a Pod moves:

```text
Volume stays behind
```

---

### Kubernetes (K8s)

Ships with:

```text
No StorageClass
```

Requires CSI drivers.

Common AWS options:

#### EBS CSI Driver

```text
ReadWriteOnce
```

Block storage.

#### EFS CSI Driver

```text
ReadWriteMany
```

Shared storage.

Benefits:

- Snapshots
- Volume expansion
- Cross-node portability

Requirements:

- IAM permissions
- CSI installation
- StorageClass configuration

---

## 6. What K3s Removes

To reduce size, K3s removes:

- In-tree cloud provider plugins
- Alpha API groups
- Built-in etcd client libraries
- Separate cloud-controller-manager process

These changes do **not** affect day-to-day Kubernetes usage.

The following work identically:

- Pods
- Deployments
- Services
- Ingress
- ConfigMaps
- Secrets
- RBAC

---

## 7. Security Defaults

### K3s

Automatically generates:

- Certificate Authority (CA)
- Cluster certificates

Uses:

```text
Single registration token
```

Defaults:

- Secrets stored in SQLite
- No encryption at rest
- Control plane runs as root

---

### Kubernetes (kubeadm)

Similar defaults, but more customization.

Examples:

#### Encrypt Secrets at Rest

```text
--encryption-provider-config
```

#### Enable Audit Logging

```text
--audit-log-path
```

Greater flexibility but more complexity.

---

## 8. HA Comparison

| Aspect | K3s HA | Kubernetes HA |
|----------|----------|----------|
| Control Plane Nodes | 3+ (odd number) | 3+ (odd number) |
| Datastore | Embedded etcd | External/Stacked etcd |
| Load Balancer | Built-in options | Must provision |
| Setup Complexity | Low | High |
| Failure Tolerance | 1 node failure (3-node cluster) | Same |
| etcd Operations | Mostly abstracted | User-managed |

### K3s HA

First node:

```bash
--cluster-init
```

Additional nodes:

```bash
K3S_URL=<server>
K3S_TOKEN=<token>
```

K3s handles etcd membership automatically.

---

### Kubernetes HA

Requires:

- etcd bootstrapping
- API server load balancer
- Control plane endpoint
- kubeadm HA configuration

Much more operational work.

---

## 9. Resource Footprint

### K3s

Typical control plane usage:

```text
~500 MB RAM
```

---

### Kubernetes (kubeadm)

Typical control plane usage:

```text
1.5–2 GB RAM
```

etcd alone often requires:

```text
~1 GB RAM
```

---

### Why K3s Exists

K3s can run comfortably on:

- Raspberry Pi
- Edge devices
- Small VMs

while still being a fully conformant Kubernetes distribution.

---

## 30-Second Mental Model

K3s is to Kubernetes what SQLite is to PostgreSQL.

| SQLite | PostgreSQL |
|----------|----------|
| Single file | Dedicated database server |
| Easy to operate | More operational complexity |
| Lower resource usage | More features |
| Simpler HA story | Enterprise-grade scalability |

Similarly:

| K3s | Kubernetes |
|----------|----------|
| Lightweight | Full-featured |
| Easy to operate | More operational overhead |
| Smaller footprint | Larger footprint |
| Same Kubernetes API | Same Kubernetes API |

### Key Takeaway

When moving from K3s to Kubernetes:

**Your manifests do not change.**

What changes is how you operate the cluster:

- Datastore management
- Backups
- Control plane recovery
- CNI selection
- Storage drivers
- Load balancer configuration
- HA architecture

The Kubernetes API remains the same; the operational model underneath is what differs.
````

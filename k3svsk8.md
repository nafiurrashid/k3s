# K3s vs Kubernetes (K8s)

## The Core Idea

K3s and Kubernetes run the same Kubernetes API and reconciliation logic.

Every manifest you write works on both:

- Pods
- Deployments
- Services
- Ingress
- ConfigMaps
- Secrets
- RBAC

The difference is mainly:

- Packaging
- Defaults
- Operational responsibilities

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

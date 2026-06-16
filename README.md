# k3s
# Stage 0 — K3s Single-Node Cluster

Now you install K3s and everything you learned maps directly onto it.

---

## Install K3s

```bash
# Install with Traefik disabled (we'll manage ingress ourselves)
# and with the metrics-server enabled

curl -sfL https://get.k3s.io | sh -s - \
  --disable traefik \
  --write-kubeconfig-mode 644

# Verify the node is ready
sudo kubectl get nodes
sudo kubectl get pods -A

# Set up kubectl for your user
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config

# Verify
kubectl get nodes
kubectl cluster-info
```

---



# Exercise 0.1 — Your First Pod (Manually)

Write a Pod spec by hand — **no Deployment yet**.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "50m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
EOF
```

---

## Watch the Pod Start

```bash
kubectl get pod nginx-pod -w
```

---

## Port Forward and Test

```bash
kubectl port-forward pod/nginx-pod 8080:80 &
```

Open another terminal:

```bash
curl http://localhost:8080
```

Expected response:

```html
<!DOCTYPE html>
<html>
...
Welcome to nginx!
...
</html>
```

---

## Inspect the Pod

### View Events and Troubleshooting Information

```bash
kubectl describe pod nginx-pod
```

Useful for debugging:

- Image pull failures
- Scheduling problems
- Probe failures
- Resource issues

---

### View the Actual Stored YAML

```bash
kubectl get pod nginx-pod -o yaml
```

Notice Kubernetes adds:

- Status information
- Pod IP
- UID
- Resource versions
- Conditions

---

## Delete the Pod

```bash
kubectl delete pod nginx-pod
```

Since this Pod is **not managed by a Deployment**, it will not be recreated.

Verify:

```bash
kubectl get pods
```

Expected:

```text
No resources found in default namespace.
```

---

## Key Learning Objectives

After completing this exercise, you should understand:

- What a Pod is
- How Kubernetes schedules Pods
- Resource requests and limits
- Liveness probes
- Readiness probes
- Port forwarding
- Viewing events and logs
- Why standalone Pods are not suitable for production
- Why Deployments exist

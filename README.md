# K8s Assignment 7 — Node Scheduling & Resource Management

## Overview
This assignment covers Kubernetes scheduling mechanisms, taints/tolerations, node affinity, QoS classes, DaemonSets, and static pods.

---

## Task 1: nodeName — Hard Pin a Pod to a Node

Bypasses the scheduler completely using `spec.nodeName`.

```yaml
spec:
  nodeName: minikube
```

```bash
kubectl apply -f static-pinned.yaml
kubectl get pod static-pinned -o wide
```

---

## Task 2: NodeSelector — Label-based Scheduling

```bash
kubectl label node minikube env=lab
kubectl apply -f selector-demo.yaml

# Remove label → second pod stays Pending
kubectl label node minikube env-
kubectl apply -f pending-selector.yaml
```

---

## Task 3: Taint Effects

| Effect | Behavior |
|---|---|
| `NoSchedule` | New pods rejected if no matching toleration |
| `PreferNoSchedule` | Soft rejection — scheduler avoids but not enforced |
| `NoExecute` | Existing pods evicted if no matching toleration |

```bash
# Add taint
kubectl taint nodes minikube test=true:NoSchedule

# Remove taint
kubectl taint nodes minikube test=true:NoSchedule-
```

---

## Task 4: Node Affinity — Required vs Preferred

| Type | Behavior |
|---|---|
| `requiredDuringScheduling` | Hard rule — pod stays Pending if not matched |
| `preferredDuringScheduling` | Soft rule — schedules even if not matched |

```bash
kubectl label node minikube disktype=ssd
kubectl apply -f affinity-demo.yaml
kubectl describe pod affinity-demo | grep -A15 Affinity
```

---

## Task 5: QoS Classes

| Class | Condition |
|---|---|
| `Guaranteed` | requests == limits for ALL containers |
| `Burstable` | requests set, limits > requests |
| `BestEffort` | No requests or limits at all |

```bash
kubectl describe pod guaranteed-pod | grep QoS
kubectl describe pod burstable-pod  | grep QoS
kubectl describe pod besteffort-pod | grep QoS
```

---

## Task 6: DaemonSet — One Pod Per Node

Ensures exactly one pod runs on every node.

```bash
kubectl get ds node-monitor
kubectl get pods -o wide | grep node-monitor
kubectl get ds -n kube-system
```

---

## Task 7: Static Pod — Managed by Kubelet

> ⚠️ `kubectl delete` won't permanently remove a static pod — you must delete the manifest file.

```bash
# Create
minikube ssh
cat > /tmp/static-nginx.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
EOF
sudo cp /tmp/static-nginx.yaml /etc/kubernetes/manifests/
exit

# Verify it comes back after deletion
kubectl delete pod static-nginx-minikube && sleep 5 && kubectl get pods | grep static-nginx

# Permanent delete
minikube ssh -- sudo rm /etc/kubernetes/manifests/static-nginx.yaml
```

---

## Key Takeaways

- **nodeName** → hardcoded, bypasses scheduler entirely
- **nodeSelector** → needs matching label on node
- **Taints/Tolerations** → node repels pods unless toleration matches
- **Node Affinity** → powerful label-based rules with hard/soft modes
- **QoS** → automatically assigned based on resource configuration
- **DaemonSet** → guaranteed one pod per node
- **Static Pod** → kubelet-managed, lives in `/etc/kubernetes/manifests/`

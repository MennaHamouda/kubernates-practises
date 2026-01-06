# Kubernetes DaemonSet with KinD (Multi-Node Cluster)

This guide demonstrates how to create a multi-node KinD cluster, deploy a DaemonSet, and limit it to specific nodes using node labels.

---

## 1. Create a Multi-Node KinD Cluster

Delete any existing KinD cluster:

```bash
kind delete cluster
```

Create a cluster configuration file:

```yaml
# cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create the cluster:

```bash
kind create cluster --config=cluster.yaml
```

Verify nodes:

```bash
kubectl get nodes
```

---

## 2. Create a DaemonSet

```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.17.1-1.0
```

Apply the DaemonSet:

```bash
kubectl apply -f daemonset.yaml
```

Check pods:

```bash
kubectl get pods -o wide
```

---

## 3. Limiting the DaemonSet to Specific Nodes

Label a node:

```bash
kubectl label nodes kind-worker gpu=true
```

Update the DaemonSet with a node selector:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        gpu: "true"
```

Apply changes:

```bash
kubectl apply -f daemonset.yaml
```

Verify scheduling:

```bash
kubectl get pods -o wide
```

---

## Summary

- DaemonSets run one pod per eligible node
- Node selectors control pod placement
- Useful for logging and monitoring workloads

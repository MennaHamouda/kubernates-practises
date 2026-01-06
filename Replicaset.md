# Kubernetes ReplicaSet – Hands-on Guide

This README explains how to create, manage, scale, and delete a **ReplicaSet** in Kubernetes using a practical Nginx example.

---

## 1. What is a ReplicaSet?

A **ReplicaSet** ensures that a specified number of identical pod replicas are running at all times. If a pod is deleted or crashes, the ReplicaSet automatically recreates it.

---

## 2. ReplicaSet Manifest

Create a file named `replicaset.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: web
          image: nginx
```

This configuration:

* Creates **3 replicas**
* Uses **nginx** as the container image
* Matches pods using the label `app: myapp`

---

## 3. Apply the ReplicaSet

Apply the manifest to the cluster:

```bash
kubectl apply -f replicaset.yaml
```

Verify that the pods were created:

```bash
kubectl get pods
```

You will notice that Kubernetes automatically generates **unique pod names**.

---

## 4. Testing Pod Recreation

If a pod is deleted, the ReplicaSet will automatically recreate it.

```bash
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods
```

You should see a **new pod with a new name** created and running.

---

## 5. Describing the ReplicaSet

To view detailed information about the ReplicaSet:

```bash
kubectl describe rs myapp
```

This shows:

* Current replicas
* Pod template
* Events and status

---

## 6. Finding Which ReplicaSet Manages a Pod

List the pods:

```bash
kubectl get pods
```

Then check which ReplicaSet owns a specific pod:

```bash
kubectl get pod <pod-name> -o=jsonpath='{.metadata.ownerReferences[0].name}'
```

---

## 7. Finding Pods Managed by a ReplicaSet

First, identify the labels used by the ReplicaSet (via `kubectl describe`).

Then list the pods using those labels:

```bash
kubectl get pods -l app=myapp
```

---

## 8. Scaling ReplicaSets

### 8.1 Imperative Scaling

Useful for quick changes (e.g., sudden traffic spikes):

```bash
kubectl scale rs myapp --replicas=4
```

Verify scaling:

```bash
kubectl get pods
```

⚠️ **Important:** Always update the manifest file afterward to avoid reverting to an old state.

---

### 8.2 Declarative Scaling

Update the `replicas` field in `replicaset.yaml`:

```yaml
replicas: 4
```

Then apply the updated manifest:

```bash
kubectl apply -f replicaset.yaml
```

---

## 9. Autoscaling Pods

Kubernetes supports multiple scaling strategies:

### Vertical Scaling

* Increase CPU or memory resources for existing pods

### Horizontal Scaling

* Increase or decrease the **number of pod replicas**

### Cluster Autoscaling

* Adds or removes **nodes** in the cluster (cloud or VM-based environments)

### Horizontal Pod Autoscaler (HPA)

To use HPA:

* Install a **metrics server**
* Kubernetes monitors CPU/memory usage
* Automatically adjusts `replicas` based on load

---

## 10. Deleting the ReplicaSet

Delete the ReplicaSet and its managed pods:

```bash
kubectl delete rs myapp
```

### Orphan the Pods

To delete the ReplicaSet **without deleting its pods**:

```bash
kubectl delete rs myapp --cascade=false
```

In this case:

* Pods become **orphaned**
* If a pod dies, it will **NOT** be recreated

---

## 11. Key Takeaways

* ReplicaSets maintain a fixed number of pods
* They automatically recreate failed or deleted pods
* Scaling can be done imperatively or declaratively
* For production workloads, **Deployments** are preferred over raw ReplicaSets

---


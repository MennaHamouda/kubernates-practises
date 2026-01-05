# Kind + NGINX Ingress Microservices Demo

This project demonstrates how to deploy a simple microservices application on a **Kubernetes cluster running on Kind**, using the **NGINX Ingress Controller** for host- and path-based routing.

---

## Architecture Overview

The application consists of **three microservices**:

| Service        | Host              | Path     | Description                           |
| -------------- | ----------------- | -------- | ------------------------------------- |
| **Main (www)** | `www.example.com` | `/`      | Serves the main website using NGINX   |
| **Admin**      | `www.example.com` | `/admin` | Admin endpoint served via echo server |
| **API**        | `api.example.com` | `/`      | API endpoint served via echo server   |

Routing is handled by **NGINX Ingress Controller** using:

* **Host-based routing**
* **Path-based routing**

---

## Prerequisites

Make sure you have the following installed:

* Docker
* Kind
* kubectl

Verify installations:

```bash
kind version
kubectl version --client
```

---

## Cluster Setup

### 1. Delete Existing Cluster (Optional)

```bash
kind delete cluster
```

### 2. Create a Kind Cluster with Ingress Support

```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

---

## Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

### Wait for the Controller to Be Ready

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

---

## Application Manifests

### www Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    app: www
spec:
  containers:
  - name: www
    image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: www
spec:
  selector:
    app: www
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

### API Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api
  labels:
    app: api
spec:
  containers:
  - name: api
    image: ealen/echo-server:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

### Admin Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: admin
  labels:
    app: admin
spec:
  containers:
  - name: admin
    image: ealen/echo-server:latest
    env:
    - name: ECHO_SERVER_BASE_PATH
      value: "/admin"
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: admin
spec:
  selector:
    app: admin
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

## Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  ingressClassName: nginx
  rules:
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: www
            port:
              number: 80
      - path: /admin
        pathType: Exact
        backend:
          service:
            name: admin
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
```

---

## Deploy the Application

Apply all manifests:

```bash
kubectl apply -f .
```

---

## Verify Resources

```bash
kubectl get pods
kubectl get svc
kubectl get ing
```

Expected ingress output:

```text
NAME      CLASS   HOSTS                             ADDRESS     PORTS   AGE
ingress   nginx   www.example.com,api.example.com   localhost   80
```

---

## Configure Local DNS

Add the following entries to `/etc/hosts`:

```text
127.0.0.1 api.example.com
127.0.0.1 www.example.com
```

---

## Testing

Open your browser and test the following URLs:

* [http://www.example.com](http://www.example.com)
* [http://www.example.com/admin](http://www.example.com/admin)
* [http://api.example.com](http://api.example.com)

### Path Behavior Notes

* `/admin` works **only** with exact match because `pathType: Exact`
* `/admin/anything` ❌ will NOT work
* `/something` on `api.example.com` ✅ works due to `pathType: Prefix`

---

## Summary

This project demonstrates:

* Running Kubernetes locally with Kind
* Installing NGINX Ingress Controller
* Host-based and path-based routing
* Using Exact vs Prefix path types

This setup is ideal for **learning Ingress fundamentals** and **local Kubernetes development**.

---

Happy Kubernetes Learning


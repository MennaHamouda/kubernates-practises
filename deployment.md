# Kubernetes Deployment Lab – Rolling Updates

## Overview

This lab demonstrates how to create and manage a Kubernetes **Deployment** using a rolling update strategy. The deployment runs an NGINX application, updates the container image, and observes how Kubernetes manages rollouts and ReplicaSets.

---

## Deployment Manifest

Create a file named **deployment.yaml** with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
  labels:
    app: sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 10
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 600
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-container
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: ENVIRONMENT
          value: "production"
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        volumeMounts:
        - name: sample-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: sample-volume
        emptyDir: {}
```

---

## Create the Deployment

Apply the deployment to the cluster:

```bash
kubectl apply -f deployment.yaml
```

---

## View Available Deployments

Verify that the deployment is running:

```bash
kubectl get deployments
```

---

## Update the Deployment Image

Edit **deployment.yaml** and change the container image to:

```yaml
image: nginx:1.27.2
```

Apply the updated deployment:

```bash
kubectl apply -f deployment.yaml
```

---

## Monitor Rollout Status

Check the rollout status of the deployment:

```bash
kubectl rollout status deployment/sample-deployment
```

---

## View ReplicaSets

List the ReplicaSets created by the deployment:

```bash
kubectl get replicasets -o wide
```

---

## Key Concepts Covered

* Kubernetes Deployments
* Rolling Update strategy
* ReplicaSets and revision history
* Liveness and Readiness probes
* Resource requests and limits

---

## Conclusion

This lab shows how Kubernetes Deployments enable controlled application updates with minimal downtime while maintaining a


# Kubernetes Rollouts, Rollbacks, and Deployment Strategies

This guide demonstrates how to manage rollouts, rollbacks, revision history, and deployment strategies in Kubernetes using kubectl.

---

## 1. Rollouts and Rollbacks

### Update Replica Count
Change the replica count to a higher number (e.g. 10) and apply the manifest:

kubectl apply -f deployment.yaml

### Pause and Resume a Rollout

kubectl rollout pause deployment sample-deployment
kubectl rollout resume deployment sample-deployment

### Rollback to the Previous ReplicaSet

kubectl rollout undo deployment sample-deployment

---

## 2. Viewing Rollout History

### Make Multiple Deployment Updates

kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.16.1"

kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.17.1"

kubectl set image deployment/nginx-deployment nginx=nginx:1.18.0
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Update to nginx 1.18.0"

### View Deployment History

kubectl rollout history deployment sample-deployment

View a specific revision:

kubectl rollout history deployment/nginx-deployment --revision=3

### Rollback to a Specific Revision

kubectl rollout undo deployment/nginx-deployment --to-revision=2

---

## 3. Using the --record Option

Create a deployment imperatively:

kubectl create deployment nginx-deployment --image=nginx:1.14.2

Record changes automatically:

kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
kubectl set image deployment/nginx-deployment nginx=nginx:1.17.1 --record
kubectl set image deployment/nginx-deployment nginx=nginx:1.18.0 --record

View history:

kubectl rollout history deployment/nginx-deployment

---

## 4. Deployment Strategies

Delete existing deployments and pods:

kubectl delete deployments --all
kubectl delete pods --all

---

## 5. Recreate Strategy

### Deployment Manifest

apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-demo
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

Apply the manifest:

kubectl apply -f recreate-deployment.yaml

Verify resources:

kubectl get deployments
kubectl get pods

Update image:

kubectl set image deployment/recreate-demo nginx=nginx:1.16.1

Monitor rollout:

kubectl rollout status deployment/recreate-demo
kubectl describe deployment recreate-demo

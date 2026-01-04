# AWS EKS Cluster and LoadBalancer Service Setup (All-in-One)

This file contains **all steps and manifests** to create an EKS cluster on AWS, deploy a pod, expose it with a LoadBalancer service, and manage the cluster.

---

## Prerequisites

- AWS CLI installed and configured
- `kubectl` installed
- `eksctl` installed
- Linux/macOS environment

---

## 1. Install `eksctl`

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

## 2. Create an EKS Cluster

```bash
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 3

aws sts get-caller-identity
```

---

## 3. Configure `kubectl` to Connect to the Cluster

```bash
kubectl version --client

aws eks update-kubeconfig --region us-east-1 --name my-cluster

kubectl get nodes
```

---

## 4. Deploy Pods

Create a Pod manifest file (`pod.yaml`) as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  containers:
    - name: web-container
      image: nginx
      ports:
        - containerPort: 80
```

Apply the pod:

```bash
kubectl apply -f pod.yaml
kubectl get pods
```

---

## 5. Create a LoadBalancer Service

Create a Service manifest file (`service.yaml`) as follows:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Apply the service:

```bash
kubectl apply -f service.yaml
kubectl get svc
```

> The LoadBalancer will provision a public IP/DNS on AWS. Use it to access your app.

---

## 6. Delete EKS Cluster

```bash
eksctl delete cluster --name my-cluster --region us-east-1
eksctl get cluster
```

---

## Notes

- Make sure your AWS IAM user has proper permissions.
- Use `kubectl describe svc web-service` to get external IP/DNS once ready.


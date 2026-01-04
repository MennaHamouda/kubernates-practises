# External Service Access via Kubernetes

This guide demonstrates how to expose an external virtual machine (VM) on AWS to a Kubernetes cluster using a **Service with no selectors** and an **EndpointSlice**, and then connect to it from a pod.

---

## Prerequisites

- A **Kubernetes cluster** up and running.
- `kubectl` configured to access your cluster.
- A private key file (`~/.ssh/id_rsa`) for accessing the EC2 instance.

---

## 1. Create an EC2 Virtual Machine on AWS

1. Log in to your AWS console.
2. Navigate to **EC2 > Instances** and launch a new Ubuntu instance.
3. Allow **SSH (port 22)** in the security group.
4. Download the key pair (`.pem`) or note the private key you will use.

---

## 2. Create a Kubernetes Service with No Selectors

Create a file called `my-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - name: ssh
      protocol: TCP
      port: 22
      targetPort: 22
```

Apply the Service to the cluster:

```bash
kubectl apply -f my-service.yaml
```

> ⚠️ Since this Service has **no selectors**, it will not automatically target any pods. We'll manually add the external endpoint in the next step.

---

## 3. Create an EndpointSlice for the External VM

Create a file called `my-endpointslice.yaml`:

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-slice
  labels:
    kubernetes.io/service-name: my-service
addressType: IPv4
endpoints:
  - addresses:
      - <EC2-PUBLIC-IP>
ports:
  - name: ssh
    port: 22
    protocol: TCP
```

Replace `<EC2-PUBLIC-IP>` with the public IP address of your EC2 instance.

Apply the EndpointSlice:

```bash
kubectl apply -f my-endpointslice.yaml
```

---

## 4. Connect from an Ubuntu Pod

Run an Ubuntu pod:

```bash
kubectl run ubuntu --image=ubuntu -it --rm -- /bin/bash
```

Inside the pod:

```bash
# Update package list
apt update

# Install SSH client
apt install -y ssh-client

# Copy your private key
mkdir -p ~/.ssh
# Use kubectl cp or any method to get the key inside the pod
# Example:
kubectl cp ~/.ssh/id_rsa ubuntu:~/.ssh/id_rsa

# Set correct permissions
chmod 600 ~/.ssh/id_rsa

# Connect to the VM via the Service
ssh ec2-user@my-service
```

> Kubernetes Service `my-service` now acts as a proxy to your external VM.

---

## Notes

- Make sure your EC2 security group allows SSH access **from the Kubernetes cluster nodes**.
- The Service does **not create pods**, it only provides a DNS name for the external VM.
- You can verify the EndpointSlice:

```bash
kubectl get endpointslice
kubectl describe endpointslice my-service-slice
```

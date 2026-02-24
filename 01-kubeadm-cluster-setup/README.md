Perfect 👏
Your structure already looks clean from the screenshot. Let’s now make your **01-kubeadm-cluster-setup/README.md** look professional, practical, and written by a real DevOps engineer — not like copied documentation.

You can paste the below directly into:

```
01-kubeadm-cluster-setup/README.md
```

---

# 01 – Kubernetes Cluster Setup using kubeadm (v1.34)

## 📌 Overview

This document explains how I built a production-style Kubernetes cluster using **kubeadm**.

The goal was to:

* Manually bootstrap a Kubernetes cluster
* Use containerd (no Docker)
* Configure networking properly
* Follow production best practices
* Prepare the cluster for version upgrades

Cluster version installed: **v1.34.x**

---

## 🧱 Architecture

* 1 Control Plane node
* 2 Worker nodes
* Ubuntu 22.04 LTS
* containerd as container runtime
* Calico as CNI

All nodes are in the same subnet.

---

## ✅ Prerequisites

| Requirement   | Details                            |
| ------------- | ---------------------------------- |
| OS            | Ubuntu 22.04 LTS                   |
| Control Plane | 2 vCPU / 4 GB RAM                  |
| Workers       | Minimum 2 vCPU recommended         |
| Networking    | All nodes reachable via private IP |
| Swap          | Disabled                           |

---

# 🔹 PART A – Common Setup (Run on ALL Nodes)

These steps must be executed on **control plane and worker nodes**.

---

## 1️⃣ Configure Kernel Modules & Networking

Kubernetes requires certain kernel modules and sysctl parameters.

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

Why this is required:

* Enables packet forwarding
* Allows iptables to see bridged traffic
* Required for Kubernetes networking to function correctly

---

## 2️⃣ Disable Swap (Mandatory)

Kubernetes will not start if swap is enabled.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

## 3️⃣ Install containerd (Container Runtime)

We are using containerd directly (no Docker).

```bash
sudo apt update
sudo apt install -y containerd
```

Generate default configuration:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Edit configuration:

```bash
sudo vi /etc/containerd/config.toml
```

Set:

```
SystemdCgroup = true
```

Restart containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

This ensures kubelet and containerd use the same cgroup driver.

---

## 4️⃣ Install Kubernetes v1.34 Components

Add official Kubernetes repository (v1.34):

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Add GPG key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add repository:

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install components:

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```

Prevent accidental upgrades:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Why hold packages?

In production, automatic upgrades can break version compatibility.
Upgrades should always be planned and controlled.

---

# 🔹 PART B – Control Plane Setup

Run these steps only on the control plane node.

---

## 1️⃣ Initialize the Cluster

```bash
sudo kubeadm init \
--apiserver-advertise-address=<CONTROL_PLANE_PRIVATE_IP> \
--pod-network-cidr=192.168.0.0/16 \
--cri-socket=unix:///run/containerd/containerd.sock
```

Important flags explained:

* `--apiserver-advertise-address` → Control plane private IP
* `--pod-network-cidr` → Must match CNI configuration
* `--cri-socket` → containerd runtime

---

## 2️⃣ Configure kubectl Access

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify:

```bash
kubectl get nodes
```

---

## 3️⃣ Install Calico CNI (Production Ready)

We use Calico for networking.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

Wait until all nodes show:

```
Ready
```

Check:

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 4️⃣ Generate Worker Join Command

```bash
kubeadm token create --print-join-command
```

Copy the output.

---

# 🔹 PART C – Worker Nodes

Run the join command (generated above) on each worker node.

Example:

```bash
sudo kubeadm join <CONTROL_PLANE_IP>:6443 \
--token <TOKEN> \
--discovery-token-ca-cert-hash sha256:<HASH>
```

---

## ✅ Final Verification

From control plane:

```bash
kubectl get nodes
```

Expected:

```
NAME                     STATUS   VERSION
control-plane-node       Ready    v1.34.x
worker-node-1            Ready    v1.34.x
worker-node-2            Ready    v1.34.x
```

---

# 🎯 Result

✔ Kubernetes cluster built manually using kubeadm
✔ containerd configured correctly
✔ Calico networking installed
✔ All nodes in Ready state
✔ Cluster prepared for controlled upgrade process

---

# 📌 Next Step

Cluster upgrade documentation is available in:

```
02-kubernetes-upgrade-v1.34-to-v1.35/
```

---

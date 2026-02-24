# 02 – Kubernetes Upgrade (v1.34 → v1.35) using kubeadm

## 📌 Overview

This document explains the step-by-step upgrade of a Kubernetes cluster from **v1.34.x to v1.35.x** using `kubeadm`.

Cluster type:

* kubeadm-based cluster
* 1 Control Plane
* 2 Worker Nodes
* containerd runtime
* Calico CNI

This upgrade was performed following **production-safe practices**, including:

* etcd snapshot backup
* kubeadm upgrade plan validation
* Rolling worker upgrade
* Version skew verification

---

# 🛡️ 0️⃣ Pre-Upgrade Checks (Control Plane)

Before touching anything in production, always validate cluster health.

## Check Cluster Status

```bash
kubectl get nodes
kubectl get pods -A
kubectl get componentstatuses
```

Check versions:

```bash
kubectl version --short
```

The cluster must be fully healthy before upgrade.

---

## 📦 Take etcd Backup (Mandatory in Production)

Even if using a single control plane, always take an etcd snapshot.

```bash
sudo ETCDCTL_API=3 etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save pre-upgrade-1.35.db
```

Verify snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot status pre-upgrade-1.35.db
```

This ensures rollback capability if anything fails.

---

# 🔺 CONTROL PLANE UPGRADE

---

## 1️⃣ Update Repository to v1.35

Edit Kubernetes repo:

```bash
sudo vi /etc/apt/sources.list.d/kubernetes.list
```

Change:

```
v1.34 → v1.35
```

Then:

```bash
sudo apt update
```

---

## 2️⃣ Upgrade kubeadm FIRST

kubeadm must always be upgraded before anything else.

Unhold (if held):

```bash
sudo apt-mark unhold kubeadm
```

Install latest patch within 1.35:

```bash
sudo apt install -y kubeadm
```

Verify:

```bash
kubeadm version
```

---

## 3️⃣ Review Upgrade Plan (Mandatory in Production)

Never skip this step.

```bash
sudo kubeadm upgrade plan
```

Carefully review:

* Target version
* etcd compatibility
* CoreDNS upgrade
* kube-proxy version
* Version skew

This step validates upgrade safety.

---

## 4️⃣ Apply Control Plane Upgrade

Use exact version shown in `upgrade plan`:

```bash
sudo kubeadm upgrade apply v1.35.x
```

This upgrades:

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* etcd (if required)
* CoreDNS
* kube-proxy

At this point:
Cluster control plane is upgraded.
Node binaries are NOT upgraded yet.

---

## 5️⃣ Upgrade kubelet + kubectl (Control Plane)

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet kubectl
sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Verify:

```bash
kubectl get nodes
```

Control plane should now show:

```
v1.35.x
```

---

# 🔻 WORKER NODE UPGRADE (Rolling Upgrade)

Upgrade one worker at a time.

---

## 1️⃣ Drain Worker (From Control Plane)

```bash
kubectl drain <worker-node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data
```

This safely evicts workloads.

---

## 2️⃣ Update Repository on Worker

```bash
sudo vi /etc/apt/sources.list.d/kubernetes.list
```

Change:

```
v1.34 → v1.35
```

Then:

```bash
sudo apt update
```

---

## 3️⃣ Upgrade kubeadm (Worker)

```bash
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm
```

Verify:

```bash
kubeadm version
```

---

## 4️⃣ Upgrade Node Configuration (Critical Step)

This is commonly skipped — and causes production issues.

```bash
sudo kubeadm upgrade node
```

This step:

* Regenerates kubelet configuration
* Aligns cluster configuration
* Updates certificates if needed

Never skip this in production.

---

## 5️⃣ Upgrade kubelet + kubectl (Worker)

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet kubectl
sudo apt-mark hold kubelet kubectl
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

## 6️⃣ Uncordon Worker

Back on control plane:

```bash
kubectl uncordon <worker-node-name>
```

Verify:

```bash
kubectl get nodes
```

Repeat same process for the next worker.

---

# ✅ Post-Upgrade Validation (Production Standard)

After all nodes are upgraded:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check system namespace:

```bash
kubectl -n kube-system get pods
```

Confirm:

* All nodes in `Ready`
* No `CrashLoopBackOff`
* No failing control plane pods
* etcd healthy
* No repeated restarts

---

# 🧠 Production Upgrade Principles

✔ Upgrade one minor version at a time
✔ Always upgrade kubeadm first
✔ Always run `kubeadm upgrade plan`
✔ Always drain worker nodes
✔ Never skip `kubeadm upgrade node`
✔ Validate cluster health before moving to next node
✔ Keep packages on hold after upgrade

---

# 🎯 Final Production Flow (Quick Summary)

### Control Plane

```
Health Check → etcd Backup → Repo Change → kubeadm Upgrade → Plan → Apply → kubelet Upgrade
```

### Worker Node

```
Drain → Repo Change → kubeadm → kubeadm upgrade node → kubelet → Uncordon
```

---

# 📌 Outcome

✔ Cluster successfully upgraded from v1.34 to v1.35
✔ etcd snapshot taken before upgrade
✔ Rolling worker upgrade performed
✔ No downtime observed
✔ Version skew maintained

---

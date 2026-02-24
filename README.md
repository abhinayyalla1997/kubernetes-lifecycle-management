# Kubernetes Lifecycle Management with kubeadm

## 📌 Overview

This repository documents the complete lifecycle of a Kubernetes cluster built using `kubeadm`.

The goal of this project was to:

* Manually bootstrap a production-style Kubernetes cluster
* Perform a controlled minor version upgrade
* Take proper etcd backups before upgrade
* Follow safe rolling upgrade practices
* Demonstrate operational discipline in cluster management

This project focuses on understanding *how Kubernetes actually works under the hood*, beyond managed services.

---

## 🧱 Cluster Architecture

* 1 Control Plane node
* 2 Worker nodes
* Ubuntu Linux
* containerd runtime
* Calico CNI
* Stacked etcd (default kubeadm setup)

All nodes communicate over private networking within the same subnet.

---

## 📂 Repository Structure

```text
kubernetes-lifecycle-management/
│
├── 01-kubeadm-cluster-setup/
│   └── Manual cluster bootstrap documentation
│
├── 02-kubernetes-upgrade-v1.34-to-v1.35/
│   └── Production-style upgrade runbook
│
├── scripts/
│   └── Optional automation helpers (backup & upgrade examples)
```

Each folder contains detailed step-by-step documentation.

---

## 🚀 1️⃣ Cluster Setup

Location:

```text
01-kubeadm-cluster-setup/
```

This section covers:

* Kernel and networking configuration
* containerd installation and configuration
* Kubernetes v1.34 installation
* kubeadm cluster initialization
* Calico CNI installation
* Worker node join process

This represents a manual production-style bootstrap using kubeadm.

---

## 🔄 2️⃣ Cluster Upgrade (v1.34 → v1.35)

Location:

```text
02-kubernetes-upgrade-v1.34-to-v1.35/
```

This section documents a safe and controlled upgrade process including:

* Pre-upgrade health validation
* etcd snapshot backup
* kubeadm upgrade plan review
* Control plane upgrade
* Rolling worker node upgrade
* Post-upgrade validation

The upgrade was performed following Kubernetes version skew rules and production best practices.

---

## 🛠️ 3️⃣ Automation Scripts (Optional)

Location:

```text
scripts/
```

This directory contains optional helper scripts for:

* Pre-upgrade cluster backup
* Upgrade automation examples

⚠ These scripts are provided for lab and learning purposes.
They must be reviewed and adjusted based on:

* Target Kubernetes version
* Cluster topology (single control plane vs HA)
* Environment requirements

The manual runbooks remain the authoritative source of truth.

---

## 🧠 What This Project Demonstrates

* Understanding of Kubernetes control plane components
* Safe upgrade sequencing using kubeadm
* etcd backup and recovery awareness
* Rolling worker upgrade strategy
* Package hold strategy to prevent accidental upgrades
* Operational discipline during cluster lifecycle management

---

## 📌 Why This Matters

In managed Kubernetes services, much of this process is abstracted away.

By performing these operations manually, we gain deeper visibility into:

* How the control plane is upgraded
* How etcd behaves during version transitions
* How kubeadm manages cluster configuration
* How node components are aligned during upgrades

This knowledge is essential for troubleshooting, architecture decisions, and production operations.


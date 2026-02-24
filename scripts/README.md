# Automation Scripts (Optional)

This directory contains optional helper scripts used during lab-based testing of:

* etcd backup
* Control plane upgrade automation

⚠ These scripts are provided as examples and **must be reviewed and adjusted** before production use.

## 1️⃣ backup-k8s-cluster.sh

This script performs a comprehensive pre-upgrade backup:

* etcd snapshot (stacked etcd)
* Kubernetes PKI and kubeconfigs
* Static pod manifests
* Cluster-wide resource export (YAML)
* Runtime and version inventory

Use case:

* Pre-upgrade backup
* Disaster recovery validation
* Lab testing

Run as root:

```bash
sudo ./backup-k8s-cluster.sh
```

---

## 2️⃣ upgrade-control-plane-example.sh

This script demonstrates how a control plane upgrade can be automated.

⚠ Important:

* Version must be updated manually inside the script
* Review minor version compatibility
* Test in non-production environment first
* Not recommended for HA clusters without modification

This script is intended for:

* Lab automation
* CI experiments
* Learning upgrade flow

---

## ⚠ Production Note

The authoritative upgrade process is documented in:

```
02-kubernetes-upgrade-v1.34-to-v1.35/
```

Always follow the documented runbook for production upgrades.

---

# 📄 What to Add in Root README.md (Small Section)

Later, in your main README, add something like:

---

## Automation

Optional helper scripts are available in the `scripts/` directory.

These scripts must be reviewed and adjusted based on:

* Target Kubernetes version
* Cluster topology (single control plane vs HA)
* Runtime configuration
* Environment (lab vs production)

Manual upgrade documentation remains the recommended approach.

---

# 🏡 Homelab

This folder stores all non-secret [patch files](https://docs.siderolabs.com/talos/v1.9/configure-your-talos-cluster/system-configuration/patching) used across my Talos Linux cluster. The purpose is to archive and keep track of every change happened on my cluster.

## Machines & Patches

Configuration details and applied patches for each node in my cluster.

### HP Mini ProDesk 400 G4
- **Node name:** `cp-1`
- **Node type:** Control plane
- **IP address:** `10.134.20.10`
- **Primary drive:** `nvme01`
- **Network interface:** `eth0`

**Applied patch files:**
- `controlplane-common.yaml`
- `install-nvme.yaml`
- `controlplane-nvme-provisioning.yaml`
- `controlplane-1.yaml`

### Lenovo Thinkcentre M70q (SMG26-275)
- **Node name:** `worker-1`
- **Node type:** Worker
- **IP address:** `10.134.20.21`
- **Primary drive:** `sda`
- **Network interface:** `eth0`

**Applied patch files:**
- `install-sata.yaml`
- `worker-sata-provisioning.yaml`
- `worker-1.yaml`

### Lenovo Thinkcentre M70q (SMG26-280)
- **Node name:** `worker-2`
- **Node type:** Worker
- **IP address:** `10.134.20.22`
- **Primary drive:** `nvme01`
- **Network interface:** `eth0`

**Applied patch files:**
- `install-nvme.yaml`
- `worker-nvme-provisioning.yaml`
- `worker-2.yaml`

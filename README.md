# proxmox-prepare

Prepare one or more Proxmox hosts for monitoring.

This repo is intended to be run from a Proxmox host and can work for:
- a single Proxmox server
- a multi-node Proxmox cluster

**Run this first**, before setting up the monitoring stack. It installs the required packages and configures each Proxmox node so that the monitoring VM can read and communicate with the metrics output from your Proxmox hosts.
If this is a cluster you will only need to do this on one host
---

## Current Scope

- Enable Ceph Prometheus module once
- Install base packages
- Install and start node exporter

## Future Scope

- smartctl exporter
- ZFS textfile collector
- Validation tasks

---

## Prerequisites

Run the following on one of your Proxmox nodes:

```bash
apt update
apt install -y git ansible jq curl
```

---

## Installation

### 1. Clone the repo

```bash
cd /opt
git clone https://github.com/45Drives/proxmox-prepare.git
cd /opt/proxmox-prepare
```

### 2. Edit the inventory

Replace `pve1`, `pve2`, `pve3` and the IPs with your environment's hostnames and IPs:

```bash
vim inventories/local/hosts.yml
```

```yaml
all:
  children:
    proxmox:
      vars:
        ansible_user: root
      hosts:
        pve1:
          ansible_host: 192.168.1.11
        pve2:
          ansible_host: 192.168.1.12
        pve3:
          ansible_host: 192.168.1.13
```

### 3. Verify connectivity

```bash
ansible proxmox -m ping
```

### 4. Run the playbook

```bash
cd /opt/proxmox-prepare
ansible-playbook site.yml
```

---

## Next Steps

Once this playbook completes, proceed to setting up the monitoring VM and running [proxmox-monitoring-stack](https://github.com/45Drives/proxmox-monitoring-stack).

You will also need to create a Prometheus user and API token on the Proxmox node. Run the following:

```bash
pveum user add prometheus@pve --comment "Prometheus monitoring user"
pveum user token add prometheus@pve monitoring --privsep 0
pveum aclmod / --users prometheus@pve --roles PVEAuditor
```

> **Save the token value that is output** — you will need it when configuring the monitoring stack.

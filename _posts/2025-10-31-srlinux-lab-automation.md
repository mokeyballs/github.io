title: "Building a SR Linux OSPF Lab on macOS with Containerlab & Ansible"
date: 2025-10-31 12:30:00 -0500
categories: [networking, automation, srlinux, containerlab, ansible]
tags: [nokia, srlinux, containerlab, ansible, devops, macos, orbstack]
description: Step-by-step build of a lightweight Nokia SR Linux OSPF lab using Containerlab, OrbStack, and Ansible automation on macOS.
---

## 🧠 Overview

This post walks through deploying a two-node Nokia SR Linux topology using **Containerlab** on macOS (Apple Silicon), validating OSPF adjacency, and preparing for Ansible-based automation.

Everything runs locally through **OrbStack**, providing native Linux environments and container networking on macOS — no external hypervisors required.

---

## 🧱 Environment

- **Host:** macOS (Apple Silicon)
- **Tools:**
  - OrbStack (Linux runtime)
  - Containerlab `v0.71.0`
  - Ansible `core 2.16.3`
  - Installed collections: `nokia.srlinux`, `ansible.netcommon`
- **Images:**
  - `ghcr.io/nokia/srlinux:24.10.2-357-arm64`

---

## ⚙️ Lab Topology

```yaml
name: srl01
topology:
  nodes:
    sr1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10.2-357-arm64
      enforce-startup-config: true
      startup-config: /home/labs/srl01/sr1.cfg
    sr2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:24.10.2-357-arm64
      enforce-startup-config: true
      startup-config: /home/labs/srl01/sr2.cfg
  links:
    - endpoints: ["sr1:ethernet-1/1", "sr2:ethernet-1/1"]
```

---

## 🔍 Deployment

```bash
sudo containerlab deploy -t srl01.clab.yml
```

Expected output:

```
╭────────────────┬─────────────────────────────────────────┬─────────┬───────────────────╮
│      Name      │                Kind/Image               │  State  │   IPv4/6 Address  │
├────────────────┼─────────────────────────────────────────┼─────────┼───────────────────┤
│ clab-srl01-sr1 │ nokia_srlinux                           │ running │ 172.20.20.2       │
│ clab-srl01-sr2 │ nokia_srlinux                           │ running │ 172.20.20.3       │
╰────────────────┴─────────────────────────────────────────┴─────────┴───────────────────╯
```

---

## 🔧 Configuration Steps

1. Enter configuration mode on each node:
   ```bash
   enter candidate
   set network-instance default protocols ospf instance 0 admin-state enable
   commit now
   save startup
   ```

2. Verify adjacency:
   ```bash
   show network-instance default protocols ospf neighbor
   ```

   Example output:
   ```
   ethernet-1/1.0 → Neighbor 1.1.1.1, State: FULL
   ```

3. Confirm persistence:
   - Redeploy with `containerlab deploy -t srl01.clab.yml`
   - OSPF forms automatically from startup configs ✅

---

## 🧩 Results

| Check | Status |
|-------|---------|
| SR Linux nodes deploy cleanly | ✅ |
| Startup configs load correctly | ✅ |
| OSPF adjacency forms automatically | ✅ |
| Ready for Ansible automation | ✅ |

---

## 🚀 Next Steps

- Build an **Ansible inventory** and playbook to validate OSPF neighbor state.
- Extend topology to include SR → EOS or IOL nodes.
- Experiment with **FastMCP 2.0** for dynamic network orchestration.

---

*Built and tested on macOS using OrbStack — no virtual machines, no overhead, just clean network automation.*

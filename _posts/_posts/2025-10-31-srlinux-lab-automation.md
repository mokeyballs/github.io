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
🔍 Deployment
bash
Copy code
sudo containerlab deploy -t srl01.clab.yml
Expected output:

arduino
Copy code
╭────────────────┬─────────────────────────────────────────┬─────────┬───────────────────╮
│      Name      │                Kind/Image               │  State  │   IPv4/6 Address  │
├────────────────┼─────────────────────────────────────────┼─────────┼───────────────────┤
│ clab-srl01-sr1 │ nokia_srlinux                           │ running │ X.X.X.X      │
│ clab-srl01-sr2 │ nokia_srlinux                           │ running │ X.X.X.X      │
╰────────────────┴─────────────────────────────────────────┴─────────┴───────────────────╯
🔧 Configuration Steps
Enter configuration mode on each node:

bash
Copy code
enter candidate
set network-instance default protocols ospf instance 0 admin-state enable
commit now
save startup
Verify adjacency:

bash
Copy code
show network-instance default protocols ospf neighbor
Example output:

sql
Copy code
ethernet-1/1.0 → Neighbor 1.1.1.1, State: FULL
Confirm persistence:

Redeploy with containerlab deploy -t srl01.clab.yml

OSPF forms automatically from startup configs ✅

🧩 Results
Check	Status
SR Linux nodes deploy cleanly	✅
Startup configs load correctly	✅
OSPF adjacency forms automatically	✅
Ready for Ansible automation	✅

🚀 Next Steps
Build an Ansible inventory and playbook to validate OSPF neighbor state.

Extend topology to include SR → EOS or IOL nodes.


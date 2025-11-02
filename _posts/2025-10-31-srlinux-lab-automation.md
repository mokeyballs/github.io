---
layout: post
title: "Nokia SR Linux OSPF Lab on macOS — Containerlab + Ansible Automation"
date: 2025-10-31 12:30:00 -0500
categories: [networking, automation, srlinux, containerlab, ansible]
tags: [nokia, srlinux, containerlab, ansible, devops, macos, orbstack]
description: A step-by-step guide to building a lightweight Nokia SR Linux OSPF lab on macOS using Containerlab, OrbStack, and Ansible automation.
---

## 🧠 Overview

This guide demonstrates how to build a **two-node Nokia SR Linux topology** using **Containerlab** on **macOS (Apple Silicon)**.  
You’ll validate **OSPF adjacency**, save startup configs, and prep the environment for **Ansible-based automation** — all without needing VMs or external hypervisors.

Everything runs locally through **OrbStack**, providing native Linux environments and full container networking.

---

## 🧱 Environment Setup

| Component | Version / Detail |
|------------|------------------|
| Host OS | macOS (Apple Silicon) |
| Linux Runtime | OrbStack |
| Containerlab | v0.71.0 |
| Ansible | core 2.16.3 |
| Collections | `nokia.srlinux`, `ansible.netcommon` |
| Image | `ghcr.io/nokia/srlinux:24.10.2-357-arm64` |

---

## ⚙️ Topology Definition

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
🚀 Deploying the Lab
Run:

bash
Copy code
sudo containerlab deploy -t srl01.clab.yml
Expected output:

arduino
Copy code

(Note: Made up random IPs)
╭────────────────┬─────────────────────────────────────────┬─────────┬───────────────────╮
│      Name      │                Kind/Image               │  State  │   IPv4/6 Address  │
├────────────────┼─────────────────────────────────────────┼─────────┼───────────────────┤
│ clab-srl01-sr1 │ nokia_srlinux                           │ running │ 172.X.X.X         │
│ clab-srl01-sr2 │ nokia_srlinux                           │ running │ 172.X.X.X         │
╰────────────────┴─────────────────────────────────────────┴─────────┴───────────────────╯
🧩 Configuring OSPF
Enter candidate mode on each SR Linux node:

bash
Copy code
enter candidate
set network-instance default protocols ospf instance 0 admin-state enable
commit now
save startup
Validate OSPF adjacency:

bash
Copy code
show network-instance default protocols ospf neighbor
Example output:

sql
Copy code
ethernet-1/1.0 → Neighbor 1.1.1.1, State: FULL
Persistence Check:

Redeploy with containerlab deploy -t srl01.clab.yml

OSPF adjacency forms automatically from startup configs ✅

✅ Validation Results
Check	Result
SR Linux nodes deploy cleanly	✅
Startup configs load correctly	✅
OSPF adjacency forms automatically	✅
Ready for Ansible integration	✅

🧭 Next Steps
Build an Ansible inventory and playbook to query OSPF neighbor state.

Add EOS or IOL nodes for multi-vendor topologies.

Experiment with FastMCP 2.0 for dynamic network orchestration.

Integrate telemetry via Prometheus + Grafana for real-time metrics.

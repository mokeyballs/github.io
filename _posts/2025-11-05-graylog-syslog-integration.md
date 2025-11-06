---
title: "Graylog + Syslog Integration Cleanup"
date: 2025-11-05
categories: [homelab, logging, graylog, syslog, cisco, truenas]
tags: [graylog, syslog-ng, truenas, cisco, homelab, troubleshooting]
description: Step-by-step breakdown of fixing duplicated syslog-ng configs and unifying all devices to Graylog over TCP/1514.
---

## 🧠 Context  
Objective: unify all infrastructure logs (firewalls, switches, router, and storage hosts) into a **Graylog** instance running in a jail on TrueNAS.  
Problem: multiple systems continued sending syslog traffic over UDP/514 even after Graylog was moved to TCP/1514, resulting in connection-refused errors and duplicate config blocks.

---

## 🧩 Issue Summary  
- Host logs repeatedly showed connection refused errors for port 514.  
- Graylog jail briefly lost its IP and web interface.  
- Legacy UDP destinations remained across several devices.  
- syslog-ng contained multiple `destination d_graylog` blocks that caused startup failure.  
- FreeNAS (backup node) was still sending to the deprecated UDP endpoint.

---

## 🧰 Actions Taken  

### 1️⃣ Graylog Jail Recovery  
- Verified jail status and network:
  ```bash
  iocage list
  sockstat -4 -l | grep -E '9000|1514|514'
  ```
  Confirmed Graylog web UI (`*:9000`) and TCP input (`*:1514`).

---

### 2️⃣ syslog-ng Fix on Main Host  
Opened `/usr/local/etc/syslog-ng.conf` and replaced the legacy UDP target with a clean TCP destination:
```bash
destination d_graylog {
    network("graylog.local" port(1514) transport("tcp"));
};

log {
    source(src);
    destination(d_graylog);
    flags(flow-control);
};
```
Validated syntax and restarted the daemon:
```bash
syslog-ng -s -f /usr/local/etc/syslog-ng.conf
service syslog-ng restart
```
✅ No syntax errors, TCP session established successfully.

---

### 3️⃣ Network Device Updates  

#### Cisco ASA (Primary / Secondary)
```bash
logging host INSIDE graylog.local tcp/1514
logging trap informational
logging device-id hostname
logging permit-hostdown
logging on
```

#### Cisco Switches (Core & Edge)
```bash
logging host graylog.local port 1514
no logging console
```

#### Small Business Switch
```bash
logging host graylog.local transport tcp port 1514
logging trap informational
```

#### Router  
- Limited to UDP only; configured:
  ```
  Remote Log Server: graylog.local
  Remote Log Port: 1026
  ```
  Confirmed messages reaching Graylog input.

---

### 4️⃣ FreeNAS (Backup Host) Cleanup  
Found three redundant `d_graylog` definitions (lines ~211, 218, 229).  
Removed duplicates and commented out old UDP references:
```bash
sed -i '' '218,237s/^/# DUP# /' /usr/local/etc/syslog-ng.conf
```
Restarted the daemon:
```bash
syslog-ng -s -f /usr/local/etc/syslog-ng.conf
service syslog-ng restart
```
Confirmed fix:
```
Syslog connection established; server='AF_INET(graylog.local:1514)'
```

Tested end-to-end:
```bash
logger -p user.info -t TEST_SYSLOG "freenas -> graylog over TCP/1514 OK"
```
Message appeared in Graylog search.

---

### 5️⃣ Validation & Snapshots  
Validated connectivity:
```bash
nc -vz graylog.local 1514
```
Checked journal and disk health:
- Journal path: `/var/db/graylog/journal`
- Disk: ~2.8 TB free, <1 % used.

Created snapshots for rollback safety:
```bash
iocage snapshot -n 2025-11-05-pre-syslog Graylog
iocage snapshot -n 2025-11-05-post-syslog Graylog
```

---

## ✅ Final Outcome  
- All infrastructure devices now send logs to Graylog over **TCP/1514**.  
- No more UDP/514 connection errors.  
- syslog-ng validated and stable on all hosts.  
- Graylog indices and retention policies confirmed (30–40 days).  
- Snapshots created for post-fix state.

---

## 🧾 Notes  
- Future improvement: create DNS entry `graylog.local` and reference that across configs.  
- Graylog journal rotation: 5 GiB / 12 h, flush = 1 M msgs or 1 min.  
- Index retention: 20 indices max, delete strategy.  
- Stable snapshot reference: `2025-11-05-post-syslog`.

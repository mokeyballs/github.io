---
title: "RCA - Proxmox Backup Server Boot Failure (Redacted)"
date: 2025-11-07
categories: [homelab, proxmox, truenas, rca]
tags: [pbs, systemd, fstab, zfs, troubleshooting]
description: Root cause analysis of a Proxmox Backup Server boot failure due to invalid fstab configuration and stale ZFS import unit.
---

# 🧩 Root Cause Analysis — Proxmox Backup Server Boot Failure

## Summary
The Proxmox Backup Server (PBS) entered **emergency mode** due to a **broken `/etc/fstab` entry** and a **stale `zfs-import@SingleDisk.service`** unit.  
The system misinterpreted a standard **ext4 volume** as a **ZFS pool**, causing dependency failures during the boot sequence.

## 🧠 Impact
- **Service:** PBS Web UI and scheduled backups unavailable until recovery.  
- **Data:** No data loss — ext4 volume intact.  
- **Scope:** Single PBS host (running on TrueNAS VM).  
- **Duration:** From boot (Nov 6, 2025) until config cleanup.

## 🔍 Symptoms
- Boot halted with:
  ```
  Failed to start zfs-import@SingleDisk.service
  Dependency failed for mnt-datastore-SingleDisk.mount
  Dependency failed for Local File Systems.
  ```
- Emergency shell prompt appeared.
- `journalctl -p err -b` showed:
  - `zfs-import@SingleDisk.service` failures
  - RRD update warnings: “time in past” (from NTP sync delay)
- Disk listing (`df -h`) confirmed healthy ext4 volume mounted at `/mnt/datastore/SingleDisk`.

## ⚙️ Root Cause
1. **Invalid `/etc/fstab` entry**
   - Duplicate mount lines for `/mnt/datastore/SingleDisk`:
     ```
     /dev/vda1  /mnt/datastore/SingleDisk  ext4  defaults  0 2
     UUID=<REDACTED>  /mnt/datastore/SingleDisk  ext4  defaults  0 2
     ```
   - The placeholder `UUID=<REDACTED>` caused `local-fs.target` to fail.

2. **Stale ZFS service**
   - Systemd attempted to import a non-existent ZFS pool named `SingleDisk`.
   - The import service persisted from earlier configuration attempts.

3. **Time sync noise**
   - RRD “time in past” messages triggered by clock drift correction (harmless).

## 🛠 Resolution
- Edited `/etc/fstab` to remove invalid placeholder line.
- Retained only valid mount entry:
  ```
  /dev/vda1  /mnt/datastore/SingleDisk  ext4  defaults  0 2
  ```
- Disabled and removed stale service:
  ```bash
  systemctl disable zfs-import@SingleDisk.service
  rm -f /etc/systemd/system/zfs-import@SingleDisk.service
  systemctl daemon-reload
  systemctl reset-failed
  ```
- Verified datastore registration:
  ```bash
  proxmox-backup-manager datastore list
  ```
- Confirmed time sync and normal service startup via:
  ```bash
  timedatectl
  systemctl --failed
  ```

## ✅ Outcome
- PBS booted normally without entering emergency mode.  
- Datastore successfully mounted and accessible.  
- No failed units remaining in `systemctl --failed`.  
- NTP active and synchronized.

## 🔒 Preventive Measures
1. **Validate `/etc/fstab` before reboot**
   ```bash
   mount -a
   systemctl --failed
   ```
2. **Use only one identifier per device** — either `UUID=` or `/dev/`, not both.
3. **Remove placeholder values** immediately after system changes.
4. **Purge obsolete units** if storage backend is reconfigured.
5. **Keep NTP active** (`timedatectl set-ntp true`) to avoid clock-related noise.
6. **Backup configuration before edits:**
   ```bash
   cp /etc/fstab /etc/fstab.bak-$(date +%F-%H%M)
   ```

## 🧾 Final Known-Good fstab
```bash
/dev/pbs/root  /               ext4  errors=remount-ro  0 1
UUID=<REDACTED> /boot/efi       vfat  defaults           0 1
/dev/pbs/swap  none            swap  sw                 0 0
proc            /proc          proc  defaults           0 0
/dev/vda1       /mnt/datastore/SingleDisk  ext4  defaults  0 2
```

## 🧩 Verification Commands
```bash
mount -a
systemctl --failed
proxmox-backup-manager datastore list
timedatectl
```

---

**Author:** JAKE
**Environment:** Proxmox Backup Server (Debian-based VM on TrueNAS BACKUP SERVER)  
**Date:** 2025-11-07  
**Status:** ✅ Resolved  

# Ansible Control Node Setup — Homelab Notes (Redacted)
*Date: 2025-11-16*

## 🧩 Overview
This document summarizes the work completed to establish a functional Ansible control node for homelab automation. All sensitive information, IP addresses, hostnames, and secret variables have been intentionally redacted or generalized for public publication.

---

## ✅ Completed Work

### **1. System Recovery & Access**
- Resolved a locked-out administrative account by entering recovery mode via the system bootloader and resetting credentials.
- Verified proper system access after reboot and confirmed administrative privileges restored.

### **2. Ansible Control Node Configuration**
- Created dedicated automation user with restricted permissions and passwordless privilege escalation.
- Validated correct home directory structure and environment variables.
- Confirmed functional Ansible installation using the system package manager.

### **3. Project Directory Structure Established**
A standardized Ansible project layout was created:

```
ansible/
├── ansible.cfg
├── inventories/
├── playbooks/
├── roles/
├── group_vars/
└── host_vars/
```

### **4. Ansible Vault Integration**
- Added encrypted `group_vars/all/vault.yml` to store sensitive values (API tokens, credentials, etc.).
- Created public-safe variable mappings in `group_vars/all/main.yml`.

### **5. Baseline Automation Role**
- Created the `common_linux` role to provide basic configuration tasks such as:
  - Installing essential packages
  - Ensuring system patches are applied
  - Setting system timezone (generalized)

### **6. Baseline Playbook Successfully Executed**
- `baseline.yml` was executed against the control node itself for validation.
- Verified:
  - Vault decryption works  
  - Sudo escalation works  
  - Inventory is functional  
  - Role executes cleanly without errors  

### **7. Inventory Structure Prepared**
A templated, expandable inventory structure was created for organizing future homelab devices.  
(Actual device names and IPs omitted intentionally.)

---

## 📌 Next Steps
- Add real homelab systems to the inventory file (redacted from this public version).  
- Deploy SSH keys to automation-managed devices.  
- Expand role and playbook library for:
  - Monitoring setup
  - Storage node maintenance
  - Virtualization host configuration
  - Network device configuration backups

## 🧠 2025-11-08 -- Cloudflare Tunnel Deployment

-   Deployed **Cloudflare Tunnel** LXC on Proxmox (Debian 12 base).
-   Registered and validated tunnel token with Cloudflare dashboard.
-   Tunnel status reported **HEALTHY** after setup.
-   Configured domain mapping for `proxmoxve.loccal.co` → internal
    Proxmox node.
-   Resolved DNS conflict and verified propagation via `nslookup`.
-   External test confirmed secure redirect (302 → Cloudflare Access
    login).
-   Added Cloudflare Zero Trust **Access Application** for Proxmox
    portal.
-   Validated authentication flow from external and LAN endpoints.

✅ **Outcome:** Secure Proxmox access through Cloudflare without any
open ports.

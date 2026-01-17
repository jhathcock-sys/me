# 🏠 My Homelab Setup

Welcome to my homelab documentation. This repository tracks the infrastructure, services, and networking configuration of my self-hosted environment. The goal is to create a resilient, automated, and organized system for home automation, media, and development.

## 🛠 Hardware Infrastructure

| Device | Role | OS / Hypervisor | Specs/Notes |
| :--- | :--- | :--- | :--- |
| **Dell R430** | Primary Server | **Proxmox VE 9** | The heavy lifter. Runs LXC containers and manages core network services. |
| **Raspberry Pi 5** | Secondary Node | **Debian (Docker)** | Dedicated low-power node for redundancy and specific lightweight services. |

---

## ☁️ Virtualization & Software Stack

### 🖥️ Node 1: Dell R430 (Proxmox VE)
The Proxmox host manages three distinct **LXC Containers** to separate concerns:

1.  **Nginx Proxy Manager (NPM)** 🛡️
    * Dedicated LXC for reverse proxying.
    * Handles SSL termination and routing for `*.home.lab`.
2.  **Pi-hole (Primary)** 🛑
    * Dedicated LXC for network-wide ad blocking and local DNS.
3.  **Docker Host** 🐳
    * A heavy-duty LXC container running the core application stack.
    * **Managed by:** [Komodo](https://komodo.io)
    * **Services Running:**
        * **Homebox:** Asset inventory and tracking.
        * **Homarr:** Main dashboard for the lab.
        * **Homepage:** Secondary status page.
        * **Minecraft Server:** Hosted game server.
        * **Syncthing:** File synchronization across devices.

### 🍓 Node 2: Raspberry Pi 5 (Docker)
Functions as a high-availability node for DNS and isolated services.

* **Mealie:** Recipe and meal planning manager.
* **Pi-hole (Secondary):** Redundant DNS to ensure uptime if the R430 is rebooting.
* **Nebula-Sync:** Automatically syncs DNS records, blocklists, and settings between the Primary (R430) and Secondary (Pi) Pi-hole instances.

---

## 🌐 Networking & DNS

* **Domain:** `home.lab` (Local only)
* **Reverse Proxy:** Nginx Proxy Manager (LXC) routes all `*.home.lab` traffic to the correct container IPs.
* **Remote Access:** **Tailscale** 🛡️ is installed on all nodes to provide secure, zero-config remote access without opening ports on the firewall.

### DNS Flow
1.  Client requests `homarr.home.lab`.
2.  **Pi-hole** resolves domain to the **NPM** IP address.
3.  **NPM** routes the request to the specific **Docker Container** port.

---

## 📊 Dashboard & Management

* **Infrastructure Management:** Proxmox Web UI & Komodo.
* **Application Dashboard:** [Homarr](https://homarr.dev) serves as the "Start Page" for the network.
* **Inventory:** Homebox tracks physical IT gear and 3D printing filaments.

---

## 🚀 Future Plans
- [ ] Migrate critical data backups to offsite storage.
- [ ] Implement additional alerting via Komodo.
- [ ] Explore automated OS patching with Ansible.
